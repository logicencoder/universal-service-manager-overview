# Universal Service Manager

**One authenticated web dashboard to start, stop, monitor, and debug dozens of long-lived apps on a Linux host — Python APIs, Node workers, bash watchers, systemd units, and Docker stacks in a single control plane.**

**Universal Service Manager (USM)** replaces scattered SSH sessions with a unified operator console. Register each app once in `services.yaml`, then manage lifecycle, tail logs, watch CPU and memory sparklines, inspect processes, control Docker Compose stacks, review boot inventory, tune UFW rules, and edit config — from browser or matching CLI (`usm start`, `usm logs`, `usm monitor`).

Built for **operators who run a private app fleet** on one machine. Password auth is optional; this overview describes product behaviour only.

**Made by [Logic Encoder](https://logicencoder.com)**

Private source: [logicencoder/universal-service-manager](https://github.com/logicencoder/universal-service-manager)

---

## What you can do

| Area | In plain language |
|------|-------------------|
| **Services grid** | Card per app with start/stop/restart, sparklines, filters, bulk actions |
| **Overview table** | Sortable PID, port, CPU, memory, uptime across fleet |
| **Resources** | Host CPU/memory/disk/network plus per-service leaderboard |
| **Processes** | htop-style list with tree view and kill |
| **Logs** | Full-screen tail with level filters and inline lifecycle controls |
| **Docker** | Containers, images, networks, volumes, Compose up/down/logs |
| **Systemd** | System and user units, enable/disable, journal viewer |
| **Startup** | Boot inventory — enabled units, cron @reboot, timers |
| **Firewall** | UFW status, rules, quick allow/deny on service ports |
| **Config** | Live `services.yaml` editor with validate and save |
| **Debug** | Health panels, alerts, restart leaders, event log |

Updates stream via **Server-Sent Events** with polling fallback.

---

## Feature examples (two per capability)

#### Unified service registry
1. One YAML file lists gas tracker, trading apps, and tunnel services with ports and env files.
2. Group tags like `blockchain` and `infra` organize cards and dropdown filters.

#### Multi-runtime process types
1. `type: python` launches FastAPI backends with venv script paths from working_dir.
2. `type: systemd` proxies nginx or user-scoped units without reimplementing systemd.

#### Lifecycle from browser or CLI
1. You click Start on a service card in the grid — PID file and log path activate.
2. You run `usm restart gas-tracker` from shell automation with the same engine.

#### Intelligent auto-restart
1. A crashed Python worker restarts automatically unless you clicked Stop.
2. `max_restarts: 10` caps runaway restart loops and surfaces alert in Debug tab.

#### Manual-stop protection
1. Stop sets intentionally_stopped so the five-second monitor does not fight you.
2. Start or restart clears the flag and resumes auto-restart eligibility.

#### Dependency awareness
1. `depends_on: [redis-service]` shows green/red chips on dependent cards.
2. Start order respects dependencies when bringing a stack up.

#### Service grid UX
1. Text search plus group dropdown plus running-only filter shows `12/16` style counts.
2. Drag-and-drop card reorder on the title bar persists in browser settings.

#### Overview operations table
1. You sort by CPU across all services during an incident.
2. Start All / Stop All / Restart All from overview header batches fleet ops.

#### Real-time dashboard feed
1. SSE stream pushes status and sparkline history without polling every endpoint.
2. Page Visibility API pauses heavy refresh when the browser tab is hidden.

#### Host resource dashboard
1. Per-core CPU grid shows load average and temperature when sensors exist.
2. Disk read/write rates plus extra mount points appear when configured.

#### Per-service performance visuals
1. Dual CPU/MEM sparklines on each card hold roughly sixty samples.
2. Inline progress bars show current CPU and memory with configurable decimals.

#### Process inspector
1. Sortable table lists PID, user, VIRT, RES, CPU%, MEM%, and command line.
2. Tree view toggle exposes parent/child relationships; kill guards USM-managed PIDs.

#### Log operations center
1. You filter ERR/WARN/INFO while tailing a selected service log file.
2. Download or copy visible log buffer for ticket attachment.

#### Systemd management
1. List system and user units with running-only filter.
2. Enable autostart on a unit and open journal snippet inline.

#### Boot and startup visibility
1. Startup tab lists all systemd units enabled at boot.
2. Surfaces cron @reboot lines and systemd timers in one inventory.

#### Docker container control
1. Start/stop/restart individual containers; batch stop all before maintenance.
2. Pull image by tag and prune stopped containers from the Docker tab.

#### Docker Compose stacks
1. Auto-scan finds compose files under configurable search paths.
2. Up/down/build/pull/restart/logs per stack with output panel.

#### Firewall management
1. Enable UFW and set default incoming/outgoing policies from the UI.
2. Quick allow rule tied to a service port from the service card context.

#### Configuration without SSH
1. Config tab reloads YAML, validates syntax, and Save & Apply writes to disk.
2. Add Service modal creates a new entry via API without manual file edit.

#### Security and operator prefs
1. Optional dashboard password sets session cookie — recommended behind VPN or reverse proxy.
2. Browser notifications fire on auto-restart; CPU/MEM threshold alerts highlight offenders.

#### Debug and event log
1. Debug tab shows restart leaders and active threshold violations.
2. Persisted events ring buffer captures lifecycle actions for post-mortem.

#### CLI parity
1. `usm status` lists the same fleet state as the Services grid.
2. `usm monitor` runs headless auto-restart loop without opening the browser.

#### Settings export
1. Export settings JSON before rebuilding the host.
2. Import restores theme, polling interval, and confirm-dialog preferences.

---

## What it does not do

- **Not** a cloud multi-tenant panel — one host, your `services.yaml`
- **Not** built-in git deploy — sync code separately, restart services from USM
- **Not** a replacement for full Docker/K8s orchestration at scale — operator fleet on one machine
- **Not** mandatory auth by default — password is optional; secure the network

Service config and logs stay on your machine — not published in this overview repo.

---

## Tech stack

| Layer | Technologies |
|-------|----------------|
| Backend | Python 3.10+, ThreadingHTTPServer, threading, subprocess |
| Monitoring | psutil — CPU, memory, disk, network, processes |
| Config | PyYAML — `services.yaml` |
| Frontend | Single `dashboard.html` — vanilla JS, CSS variables, canvas sparklines |
| Real-time | Server-Sent Events + in-memory caches |
| External | systemctl, docker, docker compose, ufw, journalctl |
| Persistence | State JSON, events ring, PID and log directories |
| CLI | `usm` wrapper — start/stop/logs/monitor/dashboard |

---

## Quick start

```bash
pip install psutil pyyaml
cp services.yaml.example services.yaml
python3 usm.py dashboard   # default http://0.0.0.0:5566
```

See `install.sh`, optional `usm.service`, and private repo README. [REPOS.md](REPOS.md).

---

## Related repositories

| Repository | Role |
|------------|------|
| [universal-service-manager](https://github.com/logicencoder/universal-service-manager) | Private application code |
| [universal-service-manager-overview](https://github.com/logicencoder/universal-service-manager-overview) | This product overview |

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
