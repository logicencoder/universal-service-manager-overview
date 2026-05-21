# USM — repository map

| Repository | Visibility | What it contains |
|------------|------------|------------------|
| **universal-service-manager-overview** (this) | Public | Product overview, per-tab what/why/who guides, screenshots (`assets/`), [FEATURES.md](FEATURES.md), [VS_PM2.md](VS_PM2.md) |
| [**universal-service-manager**](https://github.com/logicencoder/universal-service-manager) | Private | `usm.py`, `dashboard.html`, `services.yaml` (host-specific), [`README.md`](https://github.com/logicencoder/universal-service-manager/blob/main/README.md), [`ARCHITECTURE.md`](https://github.com/logicencoder/universal-service-manager/blob/main/ARCHITECTURE.md), [`REPOS.md`](https://github.com/logicencoder/universal-service-manager/blob/main/REPOS.md) |

## Naming rules (LogicEncoder)

- Code repo: `universal-service-manager` (no `-private` / `-public` suffix).
- Marketing repo: `universal-service-manager-overview` (no combined code+overview repo).
- No `README-private.md`; deep implementation docs live in **ARCHITECTURE.md** on the private repo.

## Production source of truth

- **SOL** path: `/home/sol/lojzo/universal-service-manager` — authoritative `services.yaml` and runtime.
- **WSL** copies are development only; do not treat laptop trees as production without diffing SOL.

## Cross-links

- Private README points here for public-facing screenshots and tab summaries.
- This README points to private **ARCHITECTURE.md** for API caches, auth flow, and file layout.
