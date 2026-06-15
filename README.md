# Universal Service Manager (USM)

![Universal Service Manager — services grid](assets/services-grid.png)

**Universal Service Manager** is a single-process Python control plane with a browser dashboard for running a large fleet of services on **your Linux server** — start/stop/restart, auto-recovery, live metrics, log tail, systemd control, Docker/Compose, firewall editing, and in-browser config editing without juggling SSH sessions. Homelab and production operators declare what to run once in YAML; the dashboard becomes the daily operations surface for the whole machine.

Typical fleets mix **Python APIs**, **Node frontends**, **bash watchers**, **systemd units**, and **Docker stacks** on one box — USM is the Stacer-style layer that unifies them instead of five terminals and a spreadsheet.

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

The **Services** tab is the default home screen and the view most operators live in daily. The hero screenshot shows a real production fleet: grouped cards (blockchain daemons, web apps, tunnels) with color-coded status, sparklines, and per-service actions.

### Controls bar

Across the top:

- **Refresh rate** — 0.5s through 10s; faster when debugging a crash loop, slower when you have forty tabs open.
- **Card size** — zoom in/out (pixel width) for dense homelab vs readable presentation mode.
- **Fleet summary** — aggregate CPU%, memory%, running count, stopped count, total auto-restarts since boot.
- **Bulk actions** — **Start All**, **Stop All**, **Restart Stopped**, force refresh, collapse/expand all groups, **+ Add** new service entry.
- **Filter** — text search, group dropdown, **running only** checkbox with live count.

### Card anatomy

Each card in the grid shows:

- **Name and type** — python, node, bash, systemd, exec.
- **Running / stopped badge** — green dot vs red; **intentionally stopped** badge when you halted a service manually so auto-restart does not fight you during maintenance.
- **CPU and memory sparklines** — ~60 samples from a background worker so the UI stays smooth.
- **PID, port, uptime** — copy-friendly for tickets.
- **Restart counters** — auto vs manual restarts; sudden auto-restart spikes flag flaky deps.
- **Per-card actions** — stop, kill, restart, open logs; drag handle to reorder within a group (order persists in the browser).

Group headers separate concerns — e.g. L1 nodes vs FastAPI backends vs static sites — so a 30-service host stays scannable.

## Overview table

The **Overview** tab is the spreadsheet view of the same fleet when cards feel too visual or you need to sort by resource usage.

Columns: **service name**, **type**, **PID**, **port**, **CPU%**, **memory**, **uptime**, and row-level **start / stop / restart / logs** buttons. Click column headers to sort — find the top memory hog in two clicks, or copy exact service names into Slack without scrolling a card wall.

Use Overview during incident triage (“what changed PID?”); use Services grid for day-to-day control and sparkline context.

![Overview tab — sortable fleet table](assets/overview-table.png)

## System resources

The **Resources** tab answers “is the box healthy?” — not just “is one app up?”.

**CPU section** — aggregate usage, per-core bars, load averages (1/5/15), temperature when sensors exist.

**Memory** — used/free, swap usage, pressure hints.

**Disk** — root filesystem usage, read/write rates, I/O wait.

**Network** — RX/TX throughput, connection counts, listening ports snapshot.

**Top services table** — ranks managed services by CPU% and memory with PID and uptime; sortable columns mirror Overview but scoped to resource hogs.

Refresh rate buttons (**1s / 2s / 5s / 10s / Off**) are independent from the Services tab — pin Resources on a second monitor during a deploy without speeding up the whole UI.

![Resources tab — host CPU, memory, disk, and network](assets/resources.png)

## Processes

The **Processes** tab is a browser-side **htop** for the entire host:

- Columns: **PID**, **user**, **CPU%**, **memory**, **state**, **threads**, full **command line**.
- **Text filter** and **column sort** — find `postgres` or a runaway worker instantly.
- **Highlighted rows** — processes that belong to USM-managed services stand out.
- **+ Monitor** on any PID — imports an ad-hoc process into the managed set so it appears on the Services grid without hand-editing YAML blind.

Use when a service card shows high CPU but you need the exact child process (e.g. a forked worker) before you kill the wrong thing.

![Processes tab — live process list with + Monitor](assets/processes.png)

## Log monitor

The **Logs** tab replaces SSH `tail -f` for every managed service.

**Service picker** — dropdown of all YAML entries; title updates to show which log you are reading.

**Source** — file log path from service config or **systemd journal** when the entry is a unit.

**Level filters** — ALL / ERR / WARN / INFO with color-coded lines.

**Toolbar** — search within buffer, pause auto-scroll, clear view, copy selection, download file, **Start / Stop / Restart** the selected service without switching tabs.

**Wrap and font** toggles for long JSON lines. When a deploy fails at 2 a.m., you stay in one browser tab: read the stack trace, restart, watch recovery.

![Logs tab — streaming log tail with filters](assets/logs.png)

## Docker

The **Docker** tab covers containers that run **alongside** YAML-managed processes — or instead of them when you prefer compose stacks.

- **Container list** — running/stopped, image name, CPU/memory from `docker stats`, port mappings, disk size.
- **Per-container actions** — start, stop, restart, logs, inspect.
- **Images** — list, pull, prune dangling layers.
- **Networks and volumes** — inspect attachments when a container “has no network”.
- **Compose stacks** — see which project names are up, which services belong to each stack.
- **Exec** — run a command inside a container from the browser.
- **Prune** — reclaim disk from unused objects after CI builds.

Restart a Python API from Services, then jump to Docker to bounce its Redis sidecar — same session, same auth.

![Docker tab — containers, images, and compose stacks](assets/docker.png)

## Systemd

The **Systemd** tab lists **system** and **user** units with:

- Active state, load state, description.
- **Enable / disable** autostart at boot.
- **Start / stop / restart** with confirmation on destructive actions.
- **Journal** shortcut into the Logs tab filtered to that unit.
- Filters: name search, scope (system vs user), running-only.

**+ Monitor** imports an existing unit into USM’s managed set — it then appears on the Services grid with sparklines and YAML-backed auto-restart policy. Useful for distro packages you did not write but still need in the fleet view (nginx, postgresql).

![Systemd tab — unit list with autostart toggles](assets/systemd.png)

## Startup

The **Startup** tab shows **what runs at boot** before you SSH in:

- Units with **enabled** vs **static** autostart.
- Tags by **priority** — USM-managed, crucial, optional.
- Toggle enable/disable without memorizing `systemctl enable --now` flags.

Plan maintenance windows: disable a non-critical daemon, reboot, confirm only crucial items came up, re-enable the rest from the same UI.

![Startup tab — boot autostart priorities](assets/startup.png)

## Firewall

The **Firewall** tab edits **UFW** without `ufw status numbered` in a terminal.

**Status bar** — firewall enabled/disabled toggle, default incoming/outgoing policies (deny/allow), live status badge, refresh, **Hide** sensitive ports/IPs for screen sharing, **Reset all rules** with confirmation.

**Active rules table** — sortable #, To, Action, From; filter box; edit and delete per row; reorder when rule precedence matters.

**Service ports panel** — quick allow/deny toggles per known service port discovered from your fleet.

**Add rule form** — protocol, port/range, source IP/CIDR, action; edit-in-place for typos.

**Recent deny log** — tail of blocked connection attempts for debugging “why can’t I reach port 443?”.

If UFW is not installed, the tab offers a one-click install command and output stream — common on fresh VPS images.

## Config and debug

The **Config** tab is an in-browser **`services.yaml` editor** with validation:

- Add or fix service blocks, toggle auto-restart, set dependencies and backoff.
- **Save and reload** fleet without nano over SSH.
- Syntax errors surface before you bounce production.

The **Debug** tab exposes a ring buffer of recent **start / stop / restart / kill** events with timings — post-mortem after a flap: “did USM restart it twice in ten seconds because the port was still in TIME_WAIT?”

## Settings

The **Settings** tab controls dashboard UX and safety rails:

| Area | Options |
|------|---------|
| Appearance | Theme, card density, font size, sparkline height |
| Defaults | Landing tab on load, global refresh interval |
| Alerts | CPU/memory thresholds, browser notifications on auto-restart |
| Logs tab | Default log level, wrap, font size |
| Docker | Auto-refresh interval |
| Safety | Confirm before stop/kill/restart; firewall privacy blur |
| Data | Export/import preferences JSON for a second browser or teammate |

![Settings tab — dashboard preferences and alerts](assets/settings.png)

Supported entry types in YAML include **python**, **node**, **bash**, **systemd**, and **exec** — plus Docker workloads from the Docker tab when they are not declared as YAML services. USM targets **mixed Linux fleets**: systemd, Docker, Python, Node, and bash in one UI with host metrics and firewall editing.

Private code: [universal-service-manager](https://github.com/logicencoder/universal-service-manager)

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
