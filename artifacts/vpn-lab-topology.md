# Virtual Network Lab & VPN Infrastructure — Topology

```
                   ┌─────────────────────────────────────────────────────┐
                   │                  VirtualBox Host                    │
                   │                                                     │
  [Internet]───────┼──[pfSense/Firewall VM]──────────────────────────── │
                   │         │               VPN Tunnel                  │
  [Remote User]───VPN────────┤           (WireGuard 10.8.0.0/24)        │
  (10.8.0.2)       │         │                                           │
                   │    ─────┴───────────────────────┐                  │
                   │    │                            │                  │
                   │  [VLAN 10 - DMZ]           [VLAN 20 - LAN]        │
                   │  192.168.10.0/24            192.168.20.0/24        │
                   │    │                            │                  │
                   │  [Nginx Proxy VM]          [Docker Host VM]        │
                   │  192.168.10.5              192.168.20.10           │
                   │   ▲                            │                   │
                   │   │                        [DNS Server VM]        │
                   │   │                        192.168.20.5           │
                   │   └────────────────────────────┘                  │
                   └─────────────────────────────────────────────────────┘
```

## VM Roles

| VM              | IP                | Role                           |
|-----------------|-------------------|--------------------------------|
| pfSense         | 192.168.1.1       | Gateway, firewall, VPN server  |
| Nginx Proxy     | 192.168.10.5      | Reverse proxy (DMZ)            |
| DNS Server      | 192.168.20.5      | Internal DNS, Pi-hole          |
| Docker Host     | 192.168.20.10     | Container workloads (LAN)      |
| VPN Client      | 10.8.0.2          | Remote access endpoint         |

## Security Controls

- SSH key-based auth only (password auth disabled)
- ufw rules: default deny inbound, explicit allow per service
- WireGuard pre-shared keys rotated monthly
- tcpdump captures reviewed after any config change
- nmap scans against all VMs after firewall rule updates
