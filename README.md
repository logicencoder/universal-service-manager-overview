# Universal Service Manager (USM)

**Centralized service and process manager for Linux servers** — Web dashboard, auto-restart, real-time monitoring, all in one tool.

> ⚡ **Production-ready** · 🐍 Python · 📜 Bash · ⚙️ Systemd · 🐳 Docker · 🌐 Web Dashboard

---

## 📸 Screenshots

### 🗂️ Grid View — Visual Service Cards
![Grid View](assets/screenshot-grid.png)
*Drag & drop service cards with real-time CPU%, Memory, PID, Port. Combined restart counter (auto/manual). Intentionally stopped badge.*

### 📋 Services Overview — Sortable Table
![Services Overview](assets/screenshot-overview.png)
*Clean table of all services. Sortable by name, type, PID, port, CPU, memory. Quick action buttons per row.*

### 🖥️ System Resources — Full Dashboard
![System Resources](assets/screenshot-resources.png)
*CPU full-width top row with per-core visualization (56 cores). Memory, Disk, Network, Services summary in grid below. Per-tab refresh rate control.*

### 📝 Log Monitor — Real-time Tail
![Log Monitor](assets/screenshot-logs.png)
*Real-time log tail with color coding (ERROR/WARN/INFO/DEBUG). Auto-scroll, Pause, Clear. Service selector dropdown.*

### ⚙️ Systemd Services — Enable/Disable Toggle
![Systemd](assets/screenshot-systemd.png)
*List of all systemd unit files. Autostart ON/OFF toggle button. Filter, Running only checkbox. Start/Stop/Restart/+Monitor.*

### 📊 Processes — htop in Browser
![Processes](assets/screenshot-processes.png)
*PID, USER, NI, VIRT, RES, CPU%, MEM%, Status (R/S/D/Z), Threads, Command. Sortable columns, text filter, managed processes highlighted.*

---

## 🎯 What is USM?

**Centralized service and process manager** for Linux servers. One web tool instead of `systemctl` + `htop` + `tail -f` + `pm2`.

- 🐍 **Python, Node.js, Bash, Systemd, Docker** — all service types
- 🔄 **Auto-restart** — with intentionally stopped flag (no conflicts)
- 📊 **System Resources** — CPU, RAM, disk, network in real time
- 📈 **Processes tab** — htop-like view in the browser
- 🐳 **Docker tab** — monitor and control containers
- ⚙️ **Systemd tab** — enable/disable autostart directly from UI
- 🌐 **Web dashboard** — dark theme, drag & drop, sortable tables
- 💾 **Non-blocking** — instant cache responses, scalable for multiple users

---

## ✨ Key Features

### 🔥 Reliability
- **Intentionally stopped flag** — manual Stop/Kill won't be overridden by auto-restart
- **Separate restart counters** — auto-restart vs manual-restart tracked separately
- **Auto-restart on startup** — services restart automatically when dashboard restarts
- **Page Visibility API** — polling paused when browser tab is not visible

### 🗂️ Dashboard Tabs

| Tab | Description |
|-----|-------------|
| **Grid View** | Drag & drop service cards, grid zoom, bulk actions |
| **Overview** | Sortable table, quick actions per row |
| **System Resources** | CPU (all cores), Memory, Disk, Network, Services — with per-tab refresh rate selector |
| **Log Monitor** | Real-time tail, color coding, auto-scroll |
| **Debug Events** | History of all actions with details |
| **Systemd** | Autostart toggle, filter, +Monitor button |
| **Processes** | htop-like: PID/USER/CPU/MEM/CMD, sortable, filter |
| **Docker** | Containers: Start/Stop/Restart/Logs, live CPU/Memory stats |

### 🔧 Service Types

| Type | Status |
|------|--------|
| `python` | ✅ Production-ready |
| `node` | ✅ Production-ready |
| `bash` | ✅ Production-ready |
| `systemd` | ✅ Production-ready |
| `exec` | ⚠️ Experimental |
| Docker | ✅ Via Docker tab |

### 🌐 REST API
Full REST API — `/api/status`, `/api/system`, `/api/processes`, `/api/docker`, `/api/systemd`, `/api/logs`, `/api/events` and more.

---

## 🛠️ Quick Start

```bash
# Install
./install.sh && source ~/.bashrc

# Start dashboard
service-dashboard 5566 &

# Open in browser
http://localhost:5566
```

---

## 🚧 Planned Features

| Feature | Description |
|---------|-------------|
| Add from Processes tab | +Monitor button to add any process directly to monitoring |
| Docker Compose support | Manage entire compose stacks |
| Log rotation | Automatic log file rotation |
| Notifications | Email/Discord alerts on crash |
| Auth / login | Password protection for the dashboard |

---

## 📖 More Information

- [Complete feature list](FEATURES.md)
- [Comparison with PM2](VS_PM2.md)

---

## 🔒 Repositories

| Repo | Contents |
|------|----------|
| **This (public)** | Overview, screenshots, feature list |
| **Private** | Full source code, API docs, architecture details |

---

**Made by [logicencoder](https://github.com/logicencoder)** · **Production-ready since Apr 2026**
