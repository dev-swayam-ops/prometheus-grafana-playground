# Solutions: Real-world Monitoring Labs

## Solution 1: Audit Application Instrumentation

**Query each endpoint**:
```bash
# API service
curl http://localhost:8000/metrics | head -50

# Node exporter
curl http://localhost:9100/metrics | head -50

# Output example:
# HELP http_requests_total Total HTTP requests
# TYPE http_requests_total counter
# http_requests_total{method="GET",status="200"} 1000
```

**Metrics Found**:
```
API Service:
- http_requests_total (counter)
- http_request_duration_seconds (histogram)
- http_requests_in_flight (gauge)

Database:
- mysql_queries_total (counter)
- mysql_query_duration_seconds (histogram)
- mysql_connections_active (gauge)

Infrastructure:
- node_cpu_seconds_total (counter)
- node_memory_free_bytes (gauge)
- node_disk_usage_percent (gauge)
```

**Explanation**: Auditing metrics reveals instrumentation coverage.

---

## Solution 2: Create Multi-service Dashboard

**Steps**:
1. Login to Grafana: `admin/admin`
2. Click "+" → Dashboard
3. Add Panel 1:
   - Query: `up{job="api"}`
   - Type: Stat
   - Title: "API Status"
4. Add Panel 2:
   - Query: Error rate query
   - Type: Graph
   - Title: "Error Rate"
5. Repeat for database and infrastructure
6. Save as "Multi-Service Health"

**Expected Result**: Dashboard shows all 4 service metrics.

---

## Solution 3: Implement API SLO

**SLO Document**:
```markdown
# API Service SLO

## Objective
Service shall be available 99.9% of the time per month.

## Calculation
- Month: 30 days = 2,592,000 seconds
- Availability: 99.9% = 0.999
- Error budget: (1 - 0.999) × 2,592,000 = 2,592 seconds = 43.2 minutes

## Monitoring
- SLI: up{job="api"} (1=up, 0=down)
- Target: avg_over_time(up[30d]) >= 0.999
- Alert: High burn rate (2-10x) within 6-hour window

## Actions if breached
- Priority 1: Stop new deployments
- Priority 2: Investigate root cause
- Priority 3: Implement fix
- Priority 4: Post-mortem
```

---

## Solution 4: Create Burn Rate Alerts

**Prometheus Rules**:
```yaml
groups:
  - name: api_slo_alerts
    rules:
      # High burn rate (10x) - 1h window
      - alert: APIHighBurnRate
        expr: |
          (1 - avg_over_time(up{job="api"}[1h])) / (1 - 0.999) > 10
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "API SLO burning at 10x rate"

      # Medium burn rate (2x) - 6h window
      - alert: APIMediumBurnRate
        expr: |
          (1 - avg_over_time(up{job="api"}[6h])) / (1 - 0.999) > 2
        for: 30m
        labels:
          severity: warning
```

**Testing**:
```bash
# Stop API service to trigger alert
docker stop api

# Check alert status
curl http://localhost:9090/api/v1/rules | jq '.data[].rules[] | select(.name=="APIHighBurnRate")'

# Should show: state="firing"
```

---

## Solution 5: Build Error Rate Dashboard Panel

**Panel Configuration**:
```
Query: sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))

Visualization: Graph / Time Series
Title: Request Error Rate

Threshold: 0.01 (1%)
- Red if > 1%
- Yellow if > 0.5%
- Green if <= 0.5%

Legend: Show on right
Y-axis: 0 to 0.1 (0-10%)
```

**Expected**: Graph shows error rate over time with color thresholds.

---

## Solution 6: Create Role-based Alerting Rules

**Alert Rules**:
```yaml
groups:
  - name: api_alerts
    rules:
      # Critical - Page on-call
      - alert: APIHighErrorRateCritical
        expr: sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) > 0.05
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Critical: API error rate > 5%"

      # Warning - Slack notification
      - alert: APIHighErrorRateWarning
        expr: sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) > 0.02
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Warning: API error rate > 2%"

      # Info - Dashboard only
      - alert: APIHighErrorRateInfo
        expr: sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) > 0.01
        for: 10m
        labels:
          severity: info
        annotations:
          summary: "Info: API error rate > 1%"
```

---

## Solution 7: Implement Latency SLO

**Recording Rules**:
```yaml
groups:
  - name: api_latency_slo
    interval: 30s
    rules:
      # Calculate P95 latency
      - record: api:latency:p95
        expr: |
          histogram_quantile(0.95, 
            sum(rate(http_request_duration_seconds_bucket[5m])) by (le, job)
          )

      # SLO target: 500ms (0.5 seconds)
      - record: api:latency:slo_target
        expr: 0.5

      # Burn rate: current / target
      - record: api:latency:burn_rate
        expr: api:latency:p95 / api:latency:slo_target
```

**Alert**:
```yaml
- alert: APILatencySLOBreach
  expr: api:latency:burn_rate > 1.5
  for: 5m
  labels:
    severity: warning
```

---

## Solution 8: Create Incident Response Runbook

**Runbook Template**:
```markdown
# Incident Runbook: High API Error Rate

## 1. Detection
- Alert: APIHighErrorRateCritical fires when error rate > 5%
- Severity: Critical (page on-call)
- Notification: PagerDuty, Slack, Email

## 2. Initial Response (0-5 min)
1. Acknowledge alert in PagerDuty
2. Open Grafana dashboard: "API Health"
3. Check metrics:
   - Error rate: `sum(rate(http_requests_total{status=~"5.."}[5m]))`
   - Latency: `histogram_quantile(0.95, http_request_duration_seconds)`
   - Dependencies: database, cache, external APIs

## 3. Investigation (5-10 min)
1. Check application logs: `docker logs api | tail -100`
2. Check database health: Are queries slow?
3. Check infrastructure: CPU/memory/disk pressure?
4. Correlate: When did error rate spike? What changed?

## 4. Resolution Options
**Option A: Restart service**
\`\`\`bash
docker restart api
\`\`\`

**Option B: Scale up**
\`\`\`bash
docker-compose up -d --scale api=3
\`\`\`

**Option C: Rollback deployment**
\`\`\`bash
git revert HEAD
docker-compose build api
docker-compose up -d
\`\`\`

## 5. Validation
- Error rate drops below 1%
- P95 latency < 500ms
- No more alert firing
- All checks green on dashboard

## 6. Communication
- Update status page
- Notify stakeholders
- Schedule post-mortem

## 7. Post-incident
- RCA: Root cause analysis
- Action items: Fix, monitoring, testing
- Update runbook if needed
```

---

## Solution 9: Setup Cross-service Correlation

**Dashboard Panel Configuration**:

**Panel 1: API Error Rate**
```promql
sum(rate(http_requests_total{status=~"5.."}[5m])) /
sum(rate(http_requests_total[5m]))
```

**Panel 2: Database Query Latency**
```promql
histogram_quantile(0.95, mysql_query_duration_seconds_bucket)
```

**Panel 3: Database Connections**
```promql
mysql_connections_active
```

**Observation**: When API error rate spikes, check if it correlates with:
- High query latency
- Exhausted connection pool
- Slow response times

**Example incident**:
```
10:05 - API error rate spikes to 8%
10:06 - Check dashboard: DB query latency jumped to 2s
10:07 - Check connections: 100/100 max (exhausted)
10:08 - Root cause: Slow query locking connections
10:09 - Kill slow query: `KILL QUERY PID`
10:10 - Error rate returns to 0.1%
```

---

## Solution 10: Simulate and Respond to Incident

**Step 1: Degrade Service**
```bash
# Option A: Make API slow
curl -X POST http://localhost:8000/debug/slow-mode

# Option B: Stop database
docker stop mysql

# Option C: Generate errors
for i in {1..100}; do curl http://localhost:8000/force-error; done
```

**Step 2: Monitor Alert**
```bash
# Watch Prometheus alerts
watch 'curl http://localhost:9090/api/v1/rules | jq ".data[].rules[] | select(.state==\"firing\")"'

# Check AlertManager
curl http://localhost:9093/api/v1/alerts | jq
```

**Step 3: Review Dashboard**
- Open Grafana
- Navigate to "API Health" dashboard
- Observe error rate spike
- Check infrastructure metrics for root cause

**Step 4: Execute Runbook**
1. Check application logs
2. Check database health
3. Identify root cause
4. Execute resolution

**Step 5: Verify Recovery**
```bash
# Return to normal
docker start mysql
# OR
curl -X POST http://localhost:8000/debug/normal-mode

# Verify metrics recover
curl http://localhost:9090/api/v1/query?query=up
# Should show 1 (up)
```

**Step 6: Document Incident**
```markdown
# Incident Report

**Incident**: High API Error Rate
**Start Time**: 10:05 UTC
**End Time**: 10:15 UTC
**Duration**: 10 minutes
**Error Budget Impact**: 5 minutes consumed

**Root Cause**: Database connection pool exhausted
**Trigger**: Slow query locked 100 connections
**Resolution**: Killed slow query, restarted API service
**Lessons Learned**: 
- Need better query monitoring
- Increase connection pool size
- Add slow query alerts
```

---

**All Solutions Complete ✅**
