# Exercises: Real-world Monitoring Labs

## Exercise 1: Audit Application Instrumentation (Easy)
Review what metrics are available from each service.

**Task**:
1. Start services: `docker-compose up -d`
2. Query metrics endpoints:
   - API: `curl http://localhost:8000/metrics`
   - MySQL: Check if exporter available
   - Nginx: Check metrics endpoint
3. Document 5 metrics from each service

**Validation**: List of metrics by service component.

---

## Exercise 2: Create Multi-service Dashboard (Easy)
Build dashboard showing health of entire stack.

**Task**:
1. Open Grafana: `http://localhost:3000`
2. Create new dashboard
3. Add 4 panels:
   - API availability (up metric)
   - API error rate
   - Database connections
   - Node CPU usage
4. Save dashboard

**Validation**: Dashboard loads, shows data for all services.

---

## Exercise 3: Implement API SLO (Easy)
Define SLO for API service availability.

**Task**:
Create SLO definition:
```
Service: API
SLO: 99.9% availability
Error Budget: 43.2 min/month
Monitor: up{job="api"}
Alert: if breach imminent
```

Document in text file.

**Validation**: SLO documented with calculation.

---

## Exercise 4: Create Burn Rate Alerts (Medium)
Setup alerts for SLO burn rate.

**Task**:
```yaml
- alert: APIHighBurnRate
  expr: (1 - avg_over_time(up{job="api"}[1h])) / (1 - 0.999) > 10
  for: 5m
```

Add to Prometheus rules, verify fires.

**Validation**: Alert fires when API unavailable.

---

## Exercise 5: Build Error Rate Dashboard Panel (Medium)
Create panel tracking request error rates.

**Task**:
1. Create graph panel with query:
```promql
sum(rate(http_requests_total{status=~"5.."}[5m])) /
sum(rate(http_requests_total[5m]))
```
2. Set threshold: 1% = red
3. Add to dashboard

**Validation**: Panel shows error rate with threshold.

---

## Exercise 6: Create Role-based Alerting Rules (Medium)
Setup different alert severities for different roles.

**Task**:
Create 3 alerts:
1. **Critical** (page on-call): > 5% error rate
2. **Warning** (slack): > 2% error rate
3. **Info** (dashboard): > 1% error rate

Add labels: `severity: critical/warning/info`

**Validation**: Alerts defined with appropriate severities.

---

## Exercise 7: Implement Latency SLO (Medium)
Track API latency as part of SLO.

**Task**:
```yaml
# P95 latency < 500ms
api:latency:p95: histogram_quantile(0.95, http_request_duration_seconds)

# Burn rate for latency SLO
api:latency_burn_rate: (api:latency:p95 / 0.5)
```

Add to recording rules.

**Validation**: Recording rules compute and appear in Prometheus.

---

## Exercise 8: Create Incident Response Runbook (Medium)
Document steps to respond to high error rate.

**Task**:
Create runbook with sections:
1. **Detection**: Alert fired
2. **Investigation**: Check logs, metrics
3. **Resolution**: Restart, scale, rollback
4. **Communication**: Notify stakeholders

**Validation**: Runbook documented with actionable steps.

---

## Exercise 9: Setup Cross-service Correlation (Medium)
Link API, database, and infrastructure metrics.

**Task**:
Create dashboard panel showing:
- API error rate
- Database query latency
- Database connection count
- Correlation when issues occur

**Validation**: Can identify when DB issues cause API errors.

---

## Exercise 10: Simulate and Respond to Incident (Medium)
Run full incident response workflow.

**Task**:
1. Degrade a service (slow down, increase errors)
2. Alerts fire (check Prometheus/AlertManager)
3. Review dashboards (identify root cause)
4. Execute runbook steps
5. Verify recovery
6. Document incident

**Validation**: Complete incident cycle documented.

---

**All Exercises Complete ✅**
