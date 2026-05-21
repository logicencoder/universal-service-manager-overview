# Complete Feature List

> **Universal Service Manager** — All features from most to least critical.

---

## 🔥 CRITICAL — Production Reliability

### 1. Background Worker + Cache Architecture
**Problem**: HTTP handlers called `systemctl` and `psutil` directly → blocking under multi-user load.

**Solution**:
- One daemon thread (`usm-background-worker`) pre-computes all data
- Three caches: `_status_cache`, `_system_cache`, `_processes_cache`
- HTTP handlers return cache instantly (< 1ms)
- Thread-safe: all updates under `threading.Lock()`

**Update intervals**:
- Service status: 1s (batch systemctl call)
- System stats: 2s (psutil)
- Process list: 3s (psutil.process_iter)
- Auto-restart check: 5s

### 2. Intentionally Stopped Flag
**Problem**: Manual Stop/Kill was immediately overridden by auto-restart.

**Solution**:
- Stop/Kill sets `intentionally_stopped = True`
- Auto-restart skips services with this flag
- Start/Restart clears flag → auto-restart works again
- UI shows red "⛔ stopped" badge on service card

### 3. Separate Restart Counters
- `restart_count` — auto-restart counter
- `manual_restart_count` — user-initiated restart counter
- Grid View shows: `total (Xa / Ym)` — X auto, Y manual

### 4. Auto-Restart on Startup
- When dashboard restarts, background worker checks services every 5s
- If service is not running and has no `intentionally_stopped` flag → auto-start
- Respects `auto_restart: true`, `restart_delay`, `max_restarts` from YAML

---

## 🖥️ Dashboard Tabs

### 5. Grid View
- Service cards with visual status (green/red border)
- Real-time: CPU%, Memory MB, PID, Port
- **Drag & drop reordering** — order saved to localStorage
- **Drag handle** — only `⋮⋮` dots on right trigger drag (not the whole card)
- **Combined restarts** — one box: total (auto / manual)
- **Intentionally stopped badge** — visible indicator
- **Grid zoom** — +/- buttons (200–800px)
- Bulk controls: Start All / Stop All / Restart All

### 6. Services Overview Tab
- Sortable table: name, type, PID, port, CPU, memory
- Quick actions per row: Start/Stop/Kill/Restart/Logs

### 7. System Resources
- **Layout**: CPU full-width top, 4 cards below (Memory/Disk/Network/Services)
- **CPU**: Total %, load avg (1/5/15 min), per-core mini-bars for every core
- **Memory**: Used/Total/Available GB, progress bar, swap usage
- **Disk**: Root filesystem usage, free space
- **Network**: RX/TX rate (MB/s), total RX/TX, TCP connections, active ports
- **Services summary**: Running/stopped count, progress bar
- **Per-tab refresh control**: 1s / 2s / 5s / 10s / Off buttons + last-updated timestamp

### 8. Processes Tab (htop-like)
- `/api/processes` endpoint — returns from cache (non-blocking)
- Columns: PID, USER, NI, VIRT, RES, CPU%, MEM%, Status (R/S/D/Z), Threads, Command
- **Sortable** — click any header column
- **Filter** — search by process name or user
- **Highlighted managed** — USM-monitored services highlighted
- Updated every 3s via background worker

### 9. Log Monitor
- Real-time log tail in browser (`tail -f` equivalent)
- Service selector dropdown
- Auto-scroll toggle, Pause/Resume, Clear button
- Color coding: ERROR=red, WARN=yellow, INFO=blue, DEBUG=grey

### 10. Debug Events
- Timeline of all actions (start/stop/restart/kill/auto-restart)
- Timestamp, action, service, success/fail, duration ms
- Expandable stdout/stderr for failed actions
- Ring buffer: last 10,000 events

### 11. Systemd Services
- List of all unit files with metadata
- Filter, "Running only" checkbox
- Start/Stop/Restart via systemctl
- **Autostart toggle** — ON/OFF button for `systemctl enable/disable`
- Managed indicator (which services are already in USM config)
- **+Monitor button** — add unmonitored service to YAML

### 12. Docker Containers
- `/api/docker` endpoint — `docker ps -a` + live stats via `docker stats --no-stream`
- Columns: ID, Name, Image, Status, CPU%, Memory, Ports, Size
- **Start/Stop/Restart** per container
- **Inline log panel** — `docker logs --tail 200 --timestamps`
- Filter by name/image, "Running only" checkbox
- Graceful error message when Docker is unavailable

---

## 🌐 Frontend Features

### 13. Page Visibility API
- Polling paused when browser tab is not visible
- Saves CPU and network traffic
- Instant refresh on tab return

### 14. Live Window Title
- Format: `🟢 X | 🔴 Y — Universal Service Manager`
- See running/stopped count without focusing the tab

### 15. Configurable Refresh Rates
- Global: 0.5s / 1s / 2s / 5s (applies to service status polling)
- Per-tab: System Resources has its own 1s / 2s / 5s / 10s / Off selector

### 16. Dark Theme
- GitHub-inspired colors (#0d1117, #161b22, #58a6ff)
- Sticky table headers on scroll
- Responsive auto-fill grid

---

## 🔌 API Endpoints

### Service Control
- `GET /api/status` — Cache (<1ms)
- `GET /api/system` — System stats
- `GET /api/processes` — Process list
- `POST /api/start|stop|restart?service=xxx`

### Docker
- `GET /api/docker` — Container list + live stats
- `POST /api/docker/start|stop|restart?id=xxx`
- `GET /api/docker/logs?id=xxx&n=200`

### Systemd
- `GET /api/systemd` — Service list
- `POST /api/systemd/enable|disable?unit=xxx`

### Logs / Debug
- `GET /api/logs?service=xxx`
- `GET /api/events`
- `POST /api/events/clear`

---

## ⚙️ Backend

### 17. Background Worker
- Single daemon thread
- Batch systemctl calls (one command for all services)
- Atomic cache updates under locks
- `orjson` for fast JSON serialization

### 18. State Management
- YAML config: `services.yaml`
- PID files: `~/.service_manager/pids/`
- Logs: `~/.service_manager/logs/`
- State: `state.json`
- Events: ring buffer (last 10,000 entries)

---

## 🔧 Service Types

| Type | Status | Notes |
|------|--------|-------|
| `python` | ✅ Ready | Python scripts |
| `node` | ✅ Ready | Node.js apps |
| `bash` | ✅ Ready | Bash scripts |
| `systemd` | ✅ Ready | Wrapper for systemd units |
| `exec` | ⚠️ Experimental | Arbitrary binaries — needs testing |
| Docker | ✅ Via Docker tab | No YAML entry needed |

---

## 🚧 Planned (not yet implemented)

1. **Add from Processes tab** — +Monitor button in Processes tab per process
2. **Docker YAML type** — `type: docker` in services.yaml
3. **Docker Compose support** — manage entire compose stacks
4. **Log rotation** — automatic rotation
5. **CPU/Memory limits** — resource limits per service
6. **Notifications** — email/Discord/webhook on crash
7. **Auth / login** — password protection for dashboard
