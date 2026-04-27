# AI-Powered Operational CRM — Runbook

## Architecture Overview
```
[Grafana :3002] ← metrics/alerts
      |
[REST API :8080] ← RBAC, audit log
      |
[PostgreSQL :5432] — crmdb
      |
[LLM API (external)] ← validated input only
```

## Initial Setup

### 1. Clone & configure environment
```bash
git clone <repo>
cd crm-platform
cp .env.example .env
# Edit .env: DB_PASSWORD, LLM_API_KEY, GRAFANA_PASSWORD
```

### 2. Deploy with docker-compose
```bash
docker-compose up -d
docker-compose ps   # verify all containers running
```

### 3. Initialize database schema
```bash
docker-compose exec api python manage.py migrate
docker-compose exec api python manage.py createsuperuser
```

### 4. Import Grafana dashboards
```bash
# Dashboards located in grafana/dashboards/
# Auto-provisioned via grafana/provisioning/dashboards.yml
```

## Health Checks

| Check               | Command                                           |
|---------------------|---------------------------------------------------|
| API health          | `curl http://localhost:8080/health`               |
| DB connectivity     | `docker-compose exec db pg_isready -U crm`        |
| Grafana reachable   | `curl -s http://localhost:3002/api/health`        |
| Container status    | `docker-compose ps`                               |

## Incident: LLM Null-Field Predictions

**Symptoms:** AI recommendation scores returning -1 or recommendation field is empty.

**Root cause:** Customer record with null `segment` or `email` field was passed directly to LLM API.

**Fix:**
1. Input validation layer (`api/validators.py`) checks for required fields before API call.
2. Null fields substituted with placeholder values and flagged in `ai_recommendations.score` (score < 0 = warning).
3. Alert rule in Grafana: `count(ai_recommendations where score < 0) > 5 in 10m`.

**Verification:**
```bash
docker-compose exec api python manage.py check_ai_pipeline
```

## Backup & Recovery

### Database backup (RPO: 1h)
```bash
# Automated via cron every hour:
docker-compose exec db pg_dump -U crm crmdb | gzip > /backup/crmdb_$(date +%Y%m%d_%H%M).sql.gz
# Retain last 48 backups
ls -t /backup/*.sql.gz | tail -n +49 | xargs rm -f
```

### Restore from backup
```bash
gunzip -c /backup/crmdb_<timestamp>.sql.gz | \
  docker-compose exec -T db psql -U crm crmdb
```

## Reboot Procedure (RTO: 20m)
1. `docker-compose up -d` — start all services
2. Confirm DB: `docker-compose exec db pg_isready -U crm`
3. Confirm API: `curl http://localhost:8080/health`
4. Confirm Grafana: open http://localhost:3002
5. Run post-restart connectivity test: `python tests/smoke_test.py`
