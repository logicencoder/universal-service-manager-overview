# Universal Service Manager (USM) — Public Overview

How the **private USM codebase** presents LogicEncoder's Linux service control plane. No `usm.py`, `services.yaml`, or host credentials live in this repository.

**Private implementation:** [universal-service-manager](https://github.com/logicencoder/universal-service-manager)  
**Architecture (operators):** [ARCHITECTURE.md](https://github.com/logicencoder/universal-service-manager/blob/main/ARCHITECTURE.md) on the private repo

> Production-ready control plane · Python + single-page dashboard · SOL host port **5566** (typical)

---

## Purpose

LogicEncoder runs many long-lived processes on one Linux server: Python APIs, Node helpers, bash watchers, systemd units, and Docker stacks. USM replaces a patchwork of SSH sessions (`systemctl`, `docker`, `htop`, `tail`, `ufw`) with **one authenticated web dashboard** that knows every service from `services.yaml`.

This public repo explains **what each tab does**, **why it exists**, and **who benefits**—with screenshots from a real SOL deployment.

---

## System position

```
Operator browser ──HTTP──► usm.py (ThreadingHTTPServer)
                                │
                    background worker (1–5s caches)
                                │
        ┌───────────────────────┼───────────────────────┐
        ▼                       ▼                       ▼
  services.yaml          systemctl / psutil        docker / ufw CLI
  ~/.service_manager/    journal + log files       compose stacks
```

---

## Screenshots

### Grid View — visual service cards
![Grid View](assets/screenshot-grid.png)
*Drag-and-drop cards with live CPU%, memory, PID, port, combined auto/manual restart counts, and intentionally-stopped badge.*

### Services Overview — sortable table
![Services Overview](assets/screenshot-overview.png)
*Dense table of all managed services with per-row Start/Stop/Restart/Logs.*

### System Resources — host health dashboard
![System Resources](assets/screenshot-resources.png)
*CPU (including per-core bars), memory, disk, network, and managed-service summary with per-tab refresh rate.*

### Log Monitor — live tail
![Log Monitor](assets/screenshot-logs.png)
*Color-coded tail with pause, clear, and service selector—equivalent to `tail -f` without SSH.*

### Systemd — unit control and autostart
![Systemd](assets/screenshot-systemd.png)
*Full unit list with enable/disable autostart, filters, and +Monitor to add units into USM.*

### Processes — htop in the browser
![Processes](assets/screenshot-processes.png)
*Sortable process table; managed PIDs highlighted.*

---

## Service grid (Grid View tab)

### What

The **Grid View** tab shows one card per entry in `services.yaml`: status border (running/stopped), CPU%, memory, PID, listening port, restart counters (auto vs manual), and bulk actions (Start All / Stop All / Restart All). Cards can be **reordered by drag-and-drop**; order persists in browser `localStorage`. Grid zoom scales card width for dense or sparse layouts.

### Why it exists

Operators managing **dozens of LogicEncoder services** need at-a-glance health without reading a table. Visual grouping matches how teams think about "the gas API" vs "the shop worker" rather than raw PIDs.

### Who benefits

- **On-call engineers** — spot a red card and restart in one click.  
- **Developers** — see whether their app is up after a deploy without SSH.  
- **Owners** — understand fleet health without learning `systemctl` syntax.

### How it fits

Grid data comes from `/api/status`, served from a **1-second cache** so the UI stays responsive under multiple browser tabs. Manual Stop/Kill sets **intentionally stopped** so auto-restart does not fight the operator.

---

## Services Overview tab

### What

A **sortable table** of the same managed services: name, type, PID, port, CPU%, memory, and quick action buttons per row (Start, Stop, Kill, Restart, Logs).

### Why it exists

Some operators prefer spreadsheets over cards—especially when copying names or sorting by CPU to find hot processes.

### Who benefits

- **Power users** — faster scanning when card layout is too large.  
- **Support** — copy exact service names into tickets or YAML edits.

### How it fits

Same cache as Grid View; switching tabs does not hammer the host because reads are in-memory.

---

## System Resources tab

### What

A **host-level dashboard**: aggregate CPU with per-core mini-bars, load averages, memory and swap, root disk usage, network RX/TX rates, TCP connection counts, active ports, and a summary of how many managed services are running vs stopped. Operators choose **per-tab refresh** (1s / 2s / 5s / 10s / Off).

### Why it exists

Service-level metrics are not enough when the box itself is saturated—runaway memory, full disk, or network spikes affect every app. USM surfaces `psutil` data on a **2-second background loop** so charts update smoothly without blocking HTTP threads.

### Who benefits

- **On-call** — distinguish "one bad service" from "the server is dying."  
- **Capacity planning** — watch cores and disk before adding new stacks.  
- **Developers** — correlate app restarts with CPU spikes on the same screen.

### How it fits

`/api/system` returns pre-computed sparkline history (~60 samples) for canvas graphs in `dashboard.html`.

---

## Processes tab

### What

A **browser htop**: PID, user, nice, virtual/resident memory, CPU%, memory%, state (R/S/D/Z), thread count, and command line. Columns are sortable; a text filter narrows the list. Processes that match USM-managed services are **highlighted**.

### Why it exists

Not every process is in `services.yaml`—stray workers, stuck shells, or third-party daemons need visibility. The tab answers "what is eating CPU right now?" without installing terminal tools.

### Who benefits

- **Operators** — find orphan PIDs before they OOM the host.  
- **Developers** — confirm their worker actually started (correct command line).  
- **Security-minded admins** — spot unexpected users or commands.

### How it fits

`/api/processes` is refreshed every **3 seconds** in the background worker; the UI never calls `psutil` directly from HTTP handlers.

---

## Systemd tab

### What

Lists **systemd unit files** with active state, description, and actions: Start, Stop, Restart, **Enable/Disable autostart**, and journal snippets. Filters include text search and "running only." **+Monitor** adds a unit into USM's YAML so it joins the managed fleet.

### Why it exists

Many LogicEncoder apps still run as native units while others use custom commands. Operators should not SSH for `systemctl enable` or forget which units are disabled after reboot.

### Who benefits

- **Operators** — toggle autostart before maintenance windows.  
- **Migrators** — pull legacy units under USM monitoring without hand-editing YAML blind.  
- **Developers** — test unit changes and immediately see journal errors in the UI.

### How it fits

Batch `systemctl` queries run in the worker; POST endpoints `/api/systemd/start|stop|restart|enable|disable` execute changes and log events to the debug ring buffer.

---

## Docker tab

### What

Lists **containers** (and related compose stacks): image, status, live CPU/memory from `docker stats`, ports, size. Per-container Start, Stop, Restart, and **inline log tail** (`docker logs`). Compose-aware actions can bring stacks up/down where configured.

### Why it exists

LogicEncoder ships containerized sidecars and compose-based stacks alongside bare-metal Python services. Splitting Docker into its own tab avoids overloading the service grid with non-YAML workloads.

### Who benefits

- **Operators** — restart a container without remembering container IDs.  
- **Developers** — read last 200 log lines after a failed deploy.  
- **DevOps** — see when Docker daemon is down (graceful UI message, not a hung page).

### How it fits

Docker data is cached every **5 seconds** via CLI (`docker ps`, `docker stats`)—no Python Docker SDK dependency on the server.

---

## Firewall tab

### What

Manages **UFW (Uncomplicated Firewall)** from the browser: enable/disable firewall, set default incoming/outgoing policies, add/edit/delete/reorder rules, filter the rule table, view recent deny log lines, and optional **privacy blur** for sensitive ports/IPs in screenshots or shared screens.

### Why it exists

Opening SSH solely to run `ufw allow` is slow and error-prone when USM already runs on the production host. Centralizing firewall edits next to service control keeps **network policy and process policy** in one session.

### Who benefits

- **Operators** — open a port for a new API after adding it to `services.yaml`.  
- **Security** — audit rules and defaults without memorizing `ufw` syntax.  
- **Demonstrations** — hide internal IPs when showing the dashboard remotely.

### How it fits

`/api/firewall/*` wraps `ufw` commands; if UFW is missing, the UI offers an install helper. Settings can persist "hide firewall ports & IPs" in local preferences.

---

## Config editor tab

### What

Edit **`services.yaml` in-browser**: syntax-aware editor, validate before save, add/remove service definitions via API, and reload the fleet without SSH. Keyboard shortcut **9** jumps to this tab in the dashboard.

### Why it exists

YAML on disk is the **source of truth** for what USM manages. Letting trusted operators add a new service block, fix a path, or toggle `auto_restart` reduces turnaround after deploys.

### Who benefits

- **Operators** — hot-fix a wrong `working_dir` during an incident.  
- **Developers** — prototype a new service entry and start it from the same UI.  
- **Reviewers** — validation catches broken YAML before it hits production reload.

### How it fits

`POST` handlers persist YAML and trigger reload logic; changes still belong in Git on the private repo for long-term audit— the editor is for **operational speed**, not a substitute for version control.

---

## Authentication

### What

Optional **dashboard password** protection: login form issues a session token; all `/api/*` routes check **Bearer token** or session cookie when a password is configured (`services.yaml` / `USM_PASSWORD`). Without a password, the API remains open on the bound interface (typical behind VPN or reverse proxy only).

### Why it exists

Port 5566 on a production host is powerful—anyone who can reach it could stop trading stacks or open firewall ports. Password + token gate reduces accidental exposure when the UI is not solely on localhost.

### Who benefits

- **Owners** — enforce a shared ops password on SOL.  
- **Operators** — clear login/logout flow with `/api/auth/login` and `/api/auth/logout`.  
- **Architects** — can still terminate TLS and auth at nginx while USM provides a second layer.

### How it fits

SHA-256 compare of configured password; tokens stored server-side for Bearer validation. See private **ARCHITECTURE.md** for handler order (`_check_auth` before API dispatch).

---

## Additional dashboard areas

| Area | What | Why |
|------|------|-----|
| **Log Monitor** | Tail per-service log files with ERROR/WARN/INFO coloring | Replace SSH `tail -f` during incidents |
| **Debug Events** | Ring buffer of start/stop/restart/kill with timings | Post-mortem without journal grep |
| **Settings** | Global refresh, theme, firewall privacy | Per-operator UX without editing YAML |

---

## Service types (managed fleet)

| Type | Role |
|------|------|
| `python` | FastAPI/Flask style apps with venv commands |
| `node` | Node SSR or tooling processes |
| `bash` | Watchers and glue scripts |
| `systemd` | Proxy control for native units |
| `exec` | Generic command lines (use with care) |
| Docker tab | Containers not declared as YAML services |

---

## REST API (summary)

Read paths (cached): `/api/status`, `/api/system`, `/api/processes`, `/api/docker`, `/api/systemd`, `/api/logs`, `/api/events`, `/api/firewall/status`.  
Write paths: service lifecycle, docker actions, systemd enable/disable, firewall rules, config save—**authenticated when password is set**.

Full endpoint list and thread model: private **ARCHITECTURE.md**.

---

## Quick start (evaluation)

```bash
git clone https://github.com/logicencoder/universal-service-manager  # private — access required
pip install psutil pyyaml
cp services.yaml.example services.yaml
python3 usm.py dashboard 5566
```

Public visitors use **this overview** plus screenshots; production `services.yaml` on SOL stays private.

---

## Comparison mindset

USM is **not** a Node-only process manager like PM2. It targets **mixed Linux fleets**: systemd + Docker + Python/Node/bash in one UI, with host metrics and UFW in the same tool. See [VS_PM2.md](VS_PM2.md) and [FEATURES.md](FEATURES.md) in this repo for depth.

---

## Repositories

| Repo | Contents |
|------|----------|
| **This (public)** | Overview, screenshots, feature marketing, tab guides |
| **[universal-service-manager](https://github.com/logicencoder/universal-service-manager)** | Source, ARCHITECTURE.md, production YAML |

Details: [REPOS.md](REPOS.md).

---

**Made by [logicencoder](https://github.com/logicencoder)** · Production control plane on LogicEncoder SOL infrastructure
