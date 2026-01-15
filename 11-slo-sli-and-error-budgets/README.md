# SLO, SLI, and Error Budgets

## What You'll Learn
- Service Level Indicators (SLI) - What to measure
- Service Level Objectives (SLO) - Reliability targets
- Error budgets - How much service can fail
- Burn rate - How fast error budget is consumed
- Implementing SLOs in Prometheus and Grafana

## Prerequisites
- Module 01: Prometheus Basics
- Module 03: PromQL Queries
- Module 04: Recording Rules and Alerting
- Understanding of service reliability concepts

## Key Concepts

### Service Level Indicator (SLI)
Quantifiable measure of service behavior:
- **Availability**: Percentage of time service responds
- **Latency**: Percentage of requests < threshold
- **Error rate**: Percentage of requests succeeding
- **Throughput**: Requests per second

### Service Level Objective (SLO)
Target value for SLI:
- Example: 99.9% availability (0.1% downtime allowed)
- Example: 99% of requests complete in < 500ms
- Example: Error rate < 0.1%

### Error Budget
Allowable downtime/failures in time period:
- SLO: 99.9% availability
- Monthly budget: 30 × 24 × 60 × (1 - 0.999) = 43.2 minutes
- If error budget consumed, stop releases and fix

### Burn Rate
How fast error budget is consumed:
- Normal burn: Using error budget as expected
- Alert burn: Using faster than expected (2x, 10x)
- Critical burn: Service at risk of breaching SLO

## Hands-on Lab

**Objective**: Define SLO, calculate error budget, create burn rate alerts.

### Step 1: Define SLI Metrics
In prometheus.yml or scrape config:
```yaml
# HTTP requests - Count successes
http_requests_total{job="api",status="200"}

# HTTP latency - P95 response time
histogram_quantile(0.95, http_request_duration_seconds_bucket)

# Error rate - Calculate percentage
rate(http_requests_total{status=~"5.."}[5m]) / 
rate(http_requests_total[5m])
```

### Step 2: Create SLO Definition
Define target SLI values:
```yaml
# SLO: 99.9% availability (43.2 min/month)
# SLO: 99% requests < 500ms
# SLO: Error rate < 0.1% (1 per 1000)

# SLI: Availability
up{job="api"} == 1

# SLI: Latency
histogram_quantile(0.95, http_request_duration_seconds_bucket{job="api"}) < 0.5

# SLI: Error rate
(rate(http_requests_total{job="api",status=~"5.."}[5m]) / 
rate(http_requests_total{job="api"}[5m])) < 0.001
```

### Step 3: Calculate Error Budget
In prometheus recording rule:
```yaml
groups:
  - name: slo_rules
    rules:
      # SLO: 99.9%
      - record: slo:availability:ratio
        expr: up{job="api"}
      
      # Monthly error budget (43.2 minutes in 30 days)
      - record: slo:error_budget:remaining
        expr: 1 - (0.999)
```

### Step 4: Create Burn Rate Alert
```yaml
groups:
  - name: burn_rate_alerts
    rules:
      # High burn rate (10x) - 1 hour window
      - alert: SLOBurnRateHigh
        expr: |
          (1 - avg_over_time(up{job="api"}[1h])) > 10 * (1 - 0.999)
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "{{ $labels.job }} SLO burning at 10x rate"

      # Medium burn rate (2x) - 6 hour window
      - alert: SLOBurnRateMedium
        expr: |
          (1 - avg_over_time(up{job="api"}[6h])) > 2 * (1 - 0.999)
        for: 30m
        labels:
          severity: warning
```

### Step 5: Query Error Budget Usage
```bash
# Error rate query
rate(http_requests_total{status=~"5.."}[5m]) / 
rate(http_requests_total[5m])

# Burn rate: consumed vs budget
(1 - up{job="api"}) / (1 - 0.999)

# Remaining budget
1 - (errors_so_far / error_budget_allowed)
```

### Step 6: Create Grafana Dashboard
Panel 1: Service Availability
```promql
up{job="api"}
```

Panel 2: Error Rate
```promql
rate(http_requests_total{job="api",status=~"5.."}[5m]) / 
rate(http_requests_total{job="api"}[5m])
```

Panel 3: Burn Rate vs Budget
```promql
(1 - up{job="api"}) / (1 - 0.999)
```

Panel 4: Error Budget Consumed (%)
```promql
(1 - avg_over_time(up{job="api"}[30d])) / (1 - 0.999) * 100
```

## Validation
✅ SLI metrics query correctly in Prometheus
✅ Recording rules calculate error budget
✅ Burn rate alerts fire appropriately
✅ Grafana dashboard displays all metrics
✅ Query error budget usage and verify calculation

## Cleanup
```bash
# Remove SLO recording rules from prometheus.yml
# Remove burn rate alerts from alert rules
# Remove Grafana dashboard if not needed
```

## Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| SLO too strict (99.99%) | Constant firefighting | Choose realistic target |
| SLO not aligned with users | Meaningless metric | Define SLI from user perspective |
| Burn rate window too short | False alerts | Use appropriate window (6h-30d) |
| Error budget calculation wrong | Incorrect budgets | Double-check: (1-SLO) × time period |
| No alerting on burn rate | SLO breached unexpectedly | Set up 2-tier burn rate alerts |

## Troubleshooting

**SLO query returns no data**:
```bash
# Check metric exists
curl 'http://localhost:9090/api/v1/query?query=up'

# Check time range has data
curl 'http://localhost:9090/api/v1/query_range?query=up&start=...'
```

**Burn rate alert not firing**:
- Check SLO percentage correct (0.999 not 99.9)
- Verify time windows in alert expression
- Check service is actually degrading

**Error budget calculation incorrect**:
- Verify formula: error_budget = (1 - SLO%) × seconds_in_period
- Example: 99.9% SLO = 0.1% error = 0.001 × 2592000s = 2592s/month

## Next Steps
- Module 12: Observability Best Practices (applying SLOs)
- Module 13: Troubleshooting and Debugging (incident response)
- Advanced: Multi-window burn rate alerts, error budget dashboards
