# Centralized Logging & Monitoring — Setup & Runbook

## Architecture

```
[Server A] ──rsyslog──► ┐
[Server B] ──rsyslog──► ├──► [Central Log Server] ──► [Grafana Dashboard]
[Server C] ──rsyslog──► ┘         │
                                   ▼
                              /var/log/central/
                              ├── auth.log
                              ├── syslog
                              ├── docker.log
                              └── nginx-access.log
```

## rsyslog Client Configuration

```bash
# /etc/rsyslog.d/50-remote.conf  (on each client)
*.* @192.168.20.20:514    # UDP (lower overhead)
*.* @@192.168.20.20:514   # TCP (reliable delivery)

# Filter: only send warnings and above
*.warn @192.168.20.20:514
```

## rsyslog Server Configuration

```bash
# /etc/rsyslog.conf  (on log server)
module(load="imudp")
input(type="imudp" port="514")
module(load="imtcp")
input(type="imtcp" port="514")

# Route by hostname
template(name="RemoteLogs" type="string"
  string="/var/log/central/%HOSTNAME%/%PROGRAMNAME%.log")
*.* ?RemoteLogs
```

## Log Rotation

```bash
# /etc/logrotate.d/central-logs
/var/log/central/*/*.log {
    daily
    rotate 14
    compress
    missingok
    notifempty
    sharedscripts
    postrotate
        systemctl reload rsyslog
    endscript
}
```

## Grafana Dashboard Queries (example)

```
# Error rate over time
count_over_time({job="syslog"} |= "ERROR" [5m])

# Auth failures
count_over_time({job="auth"} |= "Failed password" [1m])
```

## Restore / Verification Test

1. Stop rsyslog on client: `sudo systemctl stop rsyslog`
2. Verify log gap appears on server
3. Restart client: `sudo systemctl start rsyslog`
4. Confirm gap is filled after restart
5. Run: `logger -p local0.error "TEST: restore verification"` on client
6. Verify message appears in `/var/log/central/<hostname>/`
