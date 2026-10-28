# Host Inventory

| Host | IP | OS | Role | Access method | Key open ports |
|---|---|---|---|---|---|
| x86 | 192.168.41.1 | Ubuntu (kernel 4.15-5.19) | Wifi AP / web dashboard / control websocket | SSH (ubuntu user confirmed reachable; root denied) | 22, 53 (dnsmasq, AP gateway 10.42.0.1), 7123, 8080 (Vue dashboard), 10097 (wss, custom protocol), 45377 |
| nvidia-1 | 192.168.41.2 | Ubuntu (kernel 4.15-5.19) | LLM inference host | SSH (nvidia user) | 22, 111 (rpcbind), 11434 (Ollama — **unauthenticated, see FINDING-0001**) |
| nvidia-2 | 192.168.41.3 | Ubuntu (kernel 4.15-5.19) | Body/motor control host (telemetry + API) | SSH (nvidia user) | 22, 111 (rpcbind), 25786 (binary telemetry — **unauthenticated, see FINDING-0002**), 25787 (Uvicorn/FastAPI) |

Notes:
- x86 host has no confirmed SSH credential beyond `ubuntu` (untested for
  success) and `root` (confirmed denied). Revisit once we have a working
  login.
- rpcbind (111) open on all three hosts — unreviewed, check for NFS
  exports (`showmount -e <ip>`).
