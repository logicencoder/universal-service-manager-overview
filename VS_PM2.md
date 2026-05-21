# USM vs PM2 — Detailed Comparison

| Feature | USM | PM2 |
|---------|-----|-----|
| **Python** | ✅ Native first-class | ⚠️ Via interpreter |
| **Node.js** | ✅ Native | ✅ Native |
| **Bash** | ✅ Native first-class | ⚠️ Via exec |
| **Systemd** | ✅ Wrapper + enable/disable toggle | ❌ No |
| **Exec (binaries)** | ⚠️ Experimental | ✅ Native |
| **Docker** | ✅ Dedicated Docker tab | ❌ No |
| **Background worker (non-blocking)** | ✅ Cache pre-computation | ❌ Direct calls |
| **Intentionally stopped flag** | ✅ No auto-restart conflicts | ❌ No |
| **System Resources dashboard** | ✅ Full (CPU/Mem/Disk/Net) | ⚠️ Basic |
| **Processes tab (htop)** | ✅ Built-in | ❌ No |
| **Systemd enable/disable** | ✅ Toggle in UI | ❌ No |
| **Per-tab refresh rate** | ✅ System Resources has own rate | ❌ No |
| **Multi-user (no blocking)** | ✅ Instant cache | ⚠️ Can lag |
| **Page Visibility API** | ✅ Pause polling | ❌ No |
| **Live window title** | ✅ Shows running/stopped count | ❌ No |
| **Web dashboard** | ✅ Built-in, local | ☁️ PM2.io cloud |
| **Auto-restart** | ✅ Per-service config | ✅ Global |
| **Separate restart counters** | ✅ Auto vs Manual | ❌ No |
| **Log tail** | ✅ Built-in | ✅ pm2 logs |
| **Service dependencies** | ✅ depends_on | ❌ No |
| **Cluster mode** | ❌ No | ✅ Yes |
| **Log rotation** | ❌ Planned | ✅ Built-in |
| **CPU/Memory limits** | ❌ Planned | ✅ Yes |
| **Startup hooks** | ❌ No | ✅ Yes |
| **Cloud monitoring** | ❌ No | ✅ PM2.io |

---

## When to use USM

- 🐍 You have **Python** backends
- 📜 You need **Bash** scripts as services
- 🔧 You need a **Systemd** wrapper with enable/disable UI
- 📋 You want **unified config** for all language types
- 🌐 You want a **local** web dashboard (no cloud)
- 🔗 You need **service dependencies** (depends_on)
- 👥 You need **multi-user** access without lag
- 📊 You need **htop-like** process monitoring in the browser
- 🐳 You want **Docker container** monitoring alongside your services
- ⚡ You want **smart auto-restart** without manual stop conflicts

## When to use PM2

- 🚀 **Node.js only** applications
- 📊 You need **cluster mode** (multiple instances)
- 🔄 You need **log rotation**
- 💾 You need **CPU/Memory limits** per process
- ☁️ You want **cloud monitoring** (PM2.io)
- 🔌 You need **startup hooks** (systemd integration)

## When to use Portainer

- 🐳 Primarily **Docker** containers / Swarm / Kubernetes
- You need Docker volume, network, and image management
