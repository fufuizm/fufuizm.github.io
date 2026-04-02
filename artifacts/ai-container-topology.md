# AI-Powered Container Infrastructure — Topology Diagram

```
                        ┌─────────────────────────────────────────┐
                        │         Linux Host (low-resource)        │
                        │                                          │
  [Internet] ──────────►│  [Nginx Reverse Proxy :443]             │
                        │       │                                  │
                        │       ├──► [Portfolio UI  :3000]         │
                        │       ├──► [Monitoring     :3001]        │
                        │       └──► [API Gateway    :8080]        │
                        │                                          │
                        │  [AI Pipeline]                           │
                        │   OpenClaw / Kimi K2.5 API               │
                        │       │                                  │
                        │       ▼                                  │
                        │  [Metric Collector]──►[Decision Engine]  │
                        │   CPU/RAM/Disk/Svc    approval gate      │
                        │                           │              │
                        │                           ▼              │
                        │                   [Command Executor]     │
                        │                   (with human confirm)   │
                        │                                          │
                        │  [Git Repo] ◄── version-controlled cfg   │
                        └─────────────────────────────────────────┘
```

## Network Layout

| Service        | Port  | Protocol | Notes                       |
|----------------|-------|----------|-----------------------------|
| Nginx Proxy    | 443   | HTTPS    | TLS termination             |
| Portfolio UI   | 3000  | HTTP     | internal only               |
| Monitoring     | 3001  | HTTP     | internal only               |
| API Gateway    | 8080  | HTTP     | internal only               |
| SSH            | 22    | TCP      | key-based auth only         |

## Container Isolation

All services run in separate Docker containers connected via an internal
bridge network. No container exposes ports directly to the host except
Nginx on 443/80.
