# Universal Service Manager (USM)

**Universal Service Manager** is a single-process Python control plane with a browser dashboard for running a large fleet of services on one Linux host — start/stop/restart, auto-recovery, live metrics, log tail, systemd control, Docker/Compose, and firewall editing without juggling SSH sessions.

Private source: [logicencoder/universal-service-manager](https://github.com/logicencoder/universal-service-manager). Service definitions live in `services.yaml` in the private repo — not in this overview.

## The problem it solves

Homelab and production operators often run dozens of long-lived processes: Python APIs, Node helpers, bash watchers, systemd units, and Docker stacks. Without a control plane you memorize working directories, venvs, ports, and log paths — and crashed processes stay down until someone SSHs in.

USM centralizes lifecycle, monitoring, and housekeeping behind one **authenticated web UI** (default port **5566**).

## Service grid

**Grid View** — one card per managed service: running/stopped border, CPU%, memory, PID, port, auto vs manual restart counts, intentionally-stopped badge. Drag-and-drop card order persists locally. Bulk Start All / Stop All / Restart All.

**Services table** — dense sortable overview with per-row start/stop/restart and log tail.

Lifecycle supports `python`, `node`, `bash`, `exec`, and `systemd` entry types with `depends_on` ordering, auto-restart backoff, and a flag so manual stops are not fought by the supervisor.

## System resources

CPU (aggregate + per-core), memory, disk I/O, and network throughput with sparkline history (~60 samples) from a background worker so API responses stay instant.

## Log monitor

Color-coded live tail with pause, clear, and service selector — file logs or systemd journal from the UI, equivalent to `tail -f` without SSH.

## Systemd integration

Full unit list with enable/disable autostart, filters, and **+Monitor** to add existing units into USM’s managed set.

## Processes tab

Browser **htop-like** table: PID, user, CPU%, memory, state, command — managed PIDs highlighted.

## Docker and firewall

Control Docker containers and Compose stacks from the same dashboard. Edit **UFW** rules without a separate terminal session.

## Configuration model

Every managed service is declared in **`services.yaml`** (command, type, port, dependencies, log paths). Runtime state under `~/.service_manager/`. Full API and thread model in the private `ARCHITECTURE.md`.

## Screenshots

### Grid View
![Grid View](assets/screenshot-grid.png)

### Services overview
![Services Overview](assets/screenshot-overview.png)

### System resources
![System Resources](assets/screenshot-resources.png)

### Log monitor
![Log Monitor](assets/screenshot-logs.png)

### Systemd
![Systemd](assets/screenshot-systemd.png)

### Processes
![Processes](assets/screenshot-processes.png)

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
