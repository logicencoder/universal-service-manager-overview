# Universal Service Manager (USM)

![Universal Service Manager — services grid](assets/services-grid.png)

**Universal Service Manager** is a single-process Python control plane with a browser dashboard for running a large fleet of services on one Linux host — start/stop/restart, auto-recovery, live metrics, log tail, systemd control, Docker/Compose, firewall editing, and in-browser config editing without juggling SSH sessions. Homelab and production operators declare what to run once in YAML; the dashboard becomes the daily operations surface for the whole machine instead of memorizing working directories, virtualenvs, ports, and log paths across SSH sessions.

## Tech stack

| Layer | Technologies |
|-------|--------------|
| Control plane | Python 3, FastAPI, uvicorn, psutil, PyYAML |
| Dashboard | HTML, CSS, vanilla JavaScript (tabbed SPA) |
| Service types | Python venv apps, Node, bash, systemd units, generic exec |
| Containers | Docker CLI, Compose stack inspection, in-container exec |
| Host integration | systemd (system + user units), UFW firewall editing |
| Config | `services.yaml` — dependencies, auto-restart backoff, manual-stop respect |
| Auth | Optional dashboard password, session token, Bearer-protected write routes |
| Hosting | Self-hosted Linux server |

## Services grid

The **Services** tab is the main view: grouped cards for every managed entry in `services.yaml`. Each card shows running/stopped state, CPU and memory sparklines, PID, listening port, auto vs manual restart counts, and an intentionally-stopped badge when an operator halted a service on purpose so auto-restart does not fight them.

Cards support drag-and-drop reordering (persisted in the browser), adjustable card width for dense or sparse layouts, and bulk **Start All / Stop All / Restart All** for the visible group. Per-card actions include stop, kill, restart, and jump straight to logs. Group headers (for example blockchain daemons vs web apps) keep large fleets readable at a glance.

## Overview table

The **Overview** tab is the spreadsheet view of the same fleet: sortable columns for service name, type, PID, port, CPU%, memory, uptime, and row-level start/stop/restart/logs. Use it when you need to scan by resource usage, copy exact service names into tickets, or work through a long list faster than scrolling cards.

![Overview tab — sortable fleet table](assets/overview-table.png)

## System resources

The **Resources** tab answers “is the box healthy?” — not just “is one app up?”. Aggregate CPU with per-core bars, load averages, temperature where available, memory and swap, root disk usage with I/O rates, network RX/TX throughput, connection counts, active ports, and a live ranking of which managed services consume the most CPU and RAM.

Sparkline history (~60 samples) updates from a background worker so the UI stays responsive when multiple browser tabs are open. Refresh rate is configurable per tab (1s / 2s / 5s / 10s / off).

![Resources tab — host CPU, memory, disk, and network](assets/resources.png)

## Processes

The **Processes** tab is a browser-side **htop**: PID, user, CPU%, memory, state, thread count, and full command line with text filter and column sort. Processes that belong to USM-managed services are highlighted; unmanaged PIDs can be added to monitoring with **+ Monitor** so they join the fleet without hand-editing YAML blind.

![Processes tab — live process list with + Monitor](assets/processes.png)

## Log monitor

The **Logs** tab replaces SSH `tail -f`. Pick any managed service from a dropdown, stream file logs or systemd journal output, filter by level (all / error / warn / info), search within the buffer, pause auto-scroll, clear the view, copy, or download. Color-coded lines make incident triage faster when a deploy goes wrong at 2 a.m.

![Logs tab — streaming log tail with filters](assets/logs.png)

## Docker

The **Docker** tab covers containers outside (or alongside) YAML-managed processes: running/stopped status, image, CPU and memory from `docker stats`, port mappings, size, and per-container start/stop/restart. Inspect images, networks, volumes, and compose stacks; pull new images; run exec commands inside a container; prune unused resources — all from the same session where you restart your Python APIs.

![Docker tab — containers, images, and compose stacks](assets/docker.png)

## Systemd

The **Systemd** tab lists system and user units with active state, autostart enable/disable, start/stop/restart, and journal access. Filter by name, scope, or running-only. **+ Monitor** imports an existing unit into USM’s managed set so it appears on the Services grid with the rest of the fleet.

![Systemd tab — unit list with autostart toggles](assets/systemd.png)

## Startup

The **Startup** tab shows what runs at boot: units with enabled or static autostart, tagged by priority (USM-managed, crucial, optional). Enable or disable boot behavior without memorizing `systemctl` syntax — useful before maintenance windows or when onboarding a new daemon.

![Startup tab — boot autostart priorities](assets/startup.png)

## Firewall

The **Firewall** tab edits **UFW** rules in the browser: enable/disable the firewall, set default policies, add/edit/delete/reorder allow/deny rules, filter the table, and read recent deny log lines. Optional privacy blur hides sensitive ports and IPs when demonstrating the dashboard remotely.

## Config and debug

The **Config** tab validates and saves `services.yaml` in-browser — add or fix service blocks, toggle auto-restart, and reload the fleet without a separate editor session. The **Debug** tab exposes a ring buffer of recent start/stop/restart/kill events with timings for post-mortems.

## Settings

The **Settings** tab controls dashboard UX: theme, default tab on load, card density and font size, sparkline height, polling intervals, CPU/memory alert thresholds, log monitor defaults, Docker auto-refresh, confirmation prompts before destructive actions, browser notifications on auto-restart, and export/import of preferences as JSON.

![Settings tab — dashboard preferences and alerts](assets/settings.png)

## Managed service types

| Type | Typical use |
|------|-------------|
| `python` | FastAPI/Flask apps with venv commands |
| `node` | Node SSR or tooling processes |
| `bash` | Watchers and glue scripts |
| `systemd` | Proxy control for native units |
| `exec` | Generic command lines |

Docker workloads can live in the Docker tab even when not declared as YAML services. USM targets **mixed Linux fleets** — not a Node-only process manager like PM2. See [FEATURES.md](FEATURES.md) and [VS_PM2.md](VS_PM2.md) for depth.

Private code: [universal-service-manager](https://github.com/logicencoder/universal-service-manager)

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
