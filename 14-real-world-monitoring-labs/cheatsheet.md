# Cheatsheet: Real-world Monitoring Labs

## Multi-Service Monitoring Commands

| Command | Purpose | Example |
|---------|---------|---------|
| Query API metrics | Check API health | `curl http://localhost:8000/metrics` |
| Query DB metrics | Check database | `curl http://localhost:3306` (if exporter) |
| Check targets | All scraped services | `curl http://localhost:9090/targets` |
| Query SLI | Check SLO metric | `curl 'http://localhost:9090/api/v1/query?query=up'` |
| Check alerts | Firing alerts | `curl http://localhost:9090/api/v1/rules` |

## Essential SLO Metrics

| Metric | Formula | Alert |
|--------|---------|-------|
| **Availability** | `up{job="api"}` | Burn rate > 10x |
| **Latency P95** | `histogram_quantile(0.95, ...)` | > 500ms |
| **Error Rate** | `errors / total` | > 1% |
| **Throughput** | `rate(requests[5m])` | Drop > 50% |

## Recording Rules for SLO

**API Availability**:
```yaml
- record: api:availability:ratio
  expr: avg_over_time(up{job="api"}[5m])
```

**Request Latency**:
```yaml
- record: api:latency:p95
  expr: histogram_quantile(0.95, http_request_duration_seconds)
```

**Error Rate**:
```yaml
- record: api:error_rate:ratio
  expr: sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))
```

## Alert Severity Mapping

| Severity | Action | SLO Impact |
|----------|--------|-----------|
| **Critical** | Page on-call immediately | Imminent breach |
| **Warning** | Slack notification | Monitor closely |
| **Info** | Dashboard only | Trending worse |

## Burn Rate Alert Rules

**10x Burn (1 hour window)**:
```yaml
(1 - avg_over_time(up[1h])) / (1 - SLO) > 10
```

**2x Burn (6 hour window)**:
```yaml
(1 - avg_over_time(up[6h])) / (1 - SLO) > 2
```

## Multi-Service Dashboard Panels

| Panel | Query | Type | Purpose |
|-------|-------|------|---------|
| **API Status** | `up{job="api"}` | Stat | Service alive? |
| **Error Rate** | `api:error_rate:ratio` | Graph | Quality metric |
| **Latency** | `api:latency:p95` | Graph | User experience |
| **Dependencies** | `up{job=~"mysql\|cache\|..."}` | Stat | All dependencies up? |
| **Error Budget** | `error_budget_remaining` | Gauge | Time left to fix |

## Incident Response Workflow

```
1. DETECT
   ├─ Alert fires
   └─ Team notified

2. INVESTIGATE (0-5 min)
   ├─ Acknowledge alert
   ├─ Open dashboards
   └─ Check metrics

3. DIAGNOSE (5-15 min)
   ├─ Check logs
   ├─ Correlate services
   └─ Identify root cause

4. RESOLVE (15-30 min)
   ├─ Execute runbook
   ├─ Restart/scale/rollback
   └─ Verify recovery

5. COMMUNICATE
   ├─ Update status page
   ├─ Notify stakeholders
   └─ Schedule post-mortem

6. POST-INCIDENT
   ├─ RCA: Root cause analysis
   ├─ Action items
   └─ Update documentation
```

## Docker Commands for Lab

| Command | Purpose |
|---------|---------|
| `docker-compose up -d` | Start all services |
| `docker-compose down` | Stop all services |
| `docker logs service-name` | View service logs |
| `docker exec service-name bash` | SSH into container |
| `docker stats` | Real-time resource usage |

## Grafana Dashboard Setup

**Dashboard 1: SRE View**
- Panels: Availability, burn rate, error budget, incidents
- Audience: On-call engineers
- Update: Real-time

**Dashboard 2: Developer View**
- Panels: Latency by endpoint, error rate, trace links
- Audience: Development team
- Update: 1 minute

**Dashboard 3: Manager View**
- Panels: Uptime %, SLO status, cost, incidents this month
- Audience: Leadership
- Update: 5 minutes

## Alerting Best Practices

✅ **DO**:
- Base alerts on SLOs
- Use multiple windows (1h, 6h, 7d)
- Write clear runbooks
- Test alerts regularly
- Alert on symptoms, not causes
- Include severity levels
- Add dashboard links to alerts

❌ **DON'T**:
- Alert on every metric change
- Use thresholds without justification
- Create alerts without runbooks
- Alert on intermediate metrics
- Ignore low-severity alerts
- Set up bi-directional alerting
- Alert on metrics that are fixed

## Performance Checklist

✅ **Monitoring Health**:
- [ ] All services reporting metrics
- [ ] < 100k time series total
- [ ] Prometheus memory < 2GB
- [ ] Dashboard queries < 5s
- [ ] Alert latency < 1 minute
- [ ] Recording rules evaluated
- [ ] No scrape errors

✅ **SLO Implementation**:
- [ ] SLOs defined per service
- [ ] Error budgets calculated
- [ ] Burn rate alerts set up
- [ ] Recording rules in place
- [ ] Dashboards show SLO status
- [ ] Runbooks documented
- [ ] Team trained on SLOs

## Correlation Queries

**When API is slow**:
```promql
# Check database impact
{job="mysql"} >= 1

# Check infrastructure
{job="node"}
```

**When error rate high**:
```promql
# Check services
up{job=~"api|mysql|cache"}

# Check metrics correlate
api:error_rate:ratio and (mysql:slow_queries or node:cpu > 0.8)
```

## Common Lab Issues

| Issue | Diagnosis | Fix |
|-------|-----------|-----|
| No metrics | Services not running | `docker ps`, check logs |
| Alerts not firing | Condition not met | Query condition manually |
| Dashboards empty | Data source wrong | Check Prometheus URL |
| Slow queries | Cardinality too high | Use recording rules |
| Storage full | Too much data | Reduce retention |

## Quick Reference URLs

| Service | URL | Purpose |
|---------|-----|---------|
| Prometheus | http://localhost:9090 | Query, targets, rules |
| Grafana | http://localhost:3000 | Dashboards, alerts |
| AlertManager | http://localhost:9093 | Alert status, silences |
| API | http://localhost:8000 | Application endpoint |
| Nginx | http://localhost:8080 | Load balancer |

## SLO Calculation Reference

$$\text{Error Budget} = (1 - SLO\%) \times \text{Seconds in Period}$$

| SLO | Monthly | Weekly | Daily |
|-----|---------|--------|-------|
| 99% | 432 min | 100 min | 14.4 min |
| 99.5% | 216 min | 50 min | 7.2 min |
| 99.9% | 43.2 min | 10 min | 86.4 sec |
| 99.95% | 21.6 min | 5 min | 43.2 sec |

---

**Keep this handy for real-world monitoring setup!**
