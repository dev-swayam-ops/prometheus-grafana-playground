# Cheatsheet: SLO, SLI, and Error Budgets

## Key Definitions

| Term | Meaning | Example |
|------|---------|---------|
| **SLI** | Service Level Indicator - What to measure | 99% requests < 500ms |
| **SLO** | Service Level Objective - Target value | 99.9% availability |
| **SLA** | Service Level Agreement - Contract | 99.95% uptime guaranteed |
| **Error Budget** | Allowed failures in period | 43 min/month for 99.9% |
| **Burn Rate** | How fast budget consumed | 10x = uses 10 days budget in 1 day |

## Common SLOs

| Service Type | Recommended SLO | Error Budget/Month |
|--------------|-----------------|-------------------|
| **Internal tool** | 95% | 7.2 hours |
| **Standard web app** | 99% | 432 minutes |
| **Production API** | 99.5% | 216 minutes |
| **Critical service** | 99.9% | 43.2 minutes |
| **Mission critical** | 99.95% | 21.6 minutes |

## Error Budget Calculations

**Formula**: $(1 - SLO\%) \times \text{seconds in period}$

| SLO | Per Day | Per Month | Per Year |
|-----|---------|-----------|----------|
| 99.0% | 14.4m | 432m (7.2h) | 3.6d |
| 99.5% | 7.2m | 216m (3.6h) | 1.8d |
| 99.9% | 86.4s | 43.2m | 8.76h |
| 99.95% | 43.2s | 21.6m | 4.38h |
| 99.99% | 8.64s | 4.32m | 52.56m |

## SLI Queries

**Availability**:
```promql
up{job="api"}
# Returns: 1 (up) or 0 (down)
```

**Latency (P95)**:
```promql
histogram_quantile(0.95, 
  rate(http_request_duration_seconds_bucket{job="api"}[5m])
)
# Returns: seconds
```

**Error Rate**:
```promql
sum(rate(http_requests_total{job="api",status=~"5.."}[5m])) /
sum(rate(http_requests_total{job="api"}[5m]))
# Returns: 0.0 to 1.0
```

**Throughput**:
```promql
sum(rate(http_requests_total{job="api"}[5m]))
# Returns: requests/sec
```

## Burn Rate Formulas

**Formula**: $\frac{\text{Current Error Rate}}{\text{Allowed Error Rate}} = \frac{\text{Current Error Rate}}{1 - SLO}$

**Example (99.9% SLO)**:
```promql
# Allowed error rate = 1 - 0.999 = 0.001
# Calculate burn rate
(1 - up{job="api"}) / (1 - 0.999)
```

**Interpretation**:
- 1.0 = Budget consumed as expected
- 2.0 = Burning at 2x rate (15 days budget in 7.5 days)
- 10.0 = Burning at 10x rate (30 days budget in 3 days)

## Alert Thresholds

| Window | Rate | Action |
|--------|------|--------|
| **1 hour** | > 10x | Immediate page (critical) |
| **6 hours** | > 2x | Page (warning) |
| **1 day** | > 1.5x | Notify team (info) |
| **7 days** | > 1x | Review and discuss |

## Recording Rules for SLO

```yaml
groups:
  - name: slo
    rules:
      # Availability SLI
      - record: job:availability:ratio
        expr: avg_over_time(up{job="api"}[5m])
      
      # Error budget consumed (rolling 30d)
      - record: job:error_budget:consumed
        expr: 1 - avg_over_time(up{job="api"}[30d])
      
      # Burn rate (1h window)
      - record: job:burn_rate:1h
        expr: |
          (1 - avg_over_time(up{job="api"}[1h])) / 
          (1 - 0.999)
```

## Burn Rate Alert Rules

**High Burn Rate** (10x, 1h window):
```yaml
- alert: SLOHighBurnRate
  expr: |
    (1 - avg_over_time(up{job="api"}[1h])) / 
    (1 - 0.999) > 10
  for: 5m
  labels:
    severity: critical
```

**Medium Burn Rate** (2x, 6h window):
```yaml
- alert: SLOMediumBurnRate
  expr: |
    (1 - avg_over_time(up{job="api"}[6h])) / 
    (1 - 0.999) > 2
  for: 30m
  labels:
    severity: warning
```

**Low Burn Rate** (1x, 7d window):
```yaml
- alert: SLOLowBurnRate
  expr: |
    (1 - avg_over_time(up{job="api"}[7d])) / 
    (1 - 0.999) > 1
  for: 1h
  labels:
    severity: info
```

## Grafana Dashboard Queries

**Current Availability %**:
```promql
up{job="api"} * 100
# Shows: 100 (up) or 0 (down)
```

**30-day Availability %**:
```promql
avg_over_time(up{job="api"}[30d]) * 100
# Shows: e.g., 99.95
```

**Error Budget Remaining %**:
```promql
(
  (1 - 0.999) - (1 - avg_over_time(up{job="api"}[30d]))
) / (1 - 0.999) * 100
# Shows: 0-100%
```

**Burn Rate (1h)**:
```promql
(1 - avg_over_time(up{job="api"}[1h])) / (1 - 0.999)
# Shows: burn rate multiplier
```

## Commands

| Command | Purpose |
|---------|---------|
| Query SLI | `curl 'http://localhost:9090/api/v1/query?query=up'` |
| Check rules | `curl http://localhost:9090/rules` |
| Test alert | Trigger condition manually, check alert fires |
| Calculate budget | `(1 - SLO) × seconds_in_period` |
| Verify SLO | `avg_over_time(up[30d])` should be >= SLO |

## SLO vs SLA vs SLI

| Aspect | SLI | SLO | SLA |
|--------|-----|-----|-----|
| **Definition** | Measured metric | Target value | Contract |
| **Audience** | Internal | Internal | Customer |
| **Who sets** | Engineering | Product/Eng | Legal |
| **Examples** | 99.5% available | 99% target | 99.95% guaranteed |
| **Consequence** | Informs roadmap | Guides releases | Financial penalty |

## Best Practices

✅ **DO**:
- Choose realistic SLOs (achievable but ambitious)
- Align SLIs with user experience
- Start conservative, tighten over time
- Monitor error budget consumption
- Use multiple time windows for alerts
- Document your SLOs and SLIs
- Review SLOs quarterly

❌ **DON'T**:
- Set SLOs without talking to users
- Use only availability (include latency/errors)
- Ignore error budget (it will run out!)
- Alert on every metric change
- Set identical SLOs for all services
- Forget to account for maintenance
- Assume 99.99% is achievable

## Decision Matrix

**Choose SLO based on**:

| Factor | 95% | 99% | 99.5% | 99.9% |
|--------|-----|-----|-------|-------|
| **User criticality** | Low | Medium | High | Critical |
| **Team size** | Small | Medium | Large | Very Large |
| **On-call** | No | Maybe | Yes | Yes |
| **Budget/month** | 7.2h | 432m | 216m | 43m |
| **Realistic?** | ✓ | ✓ | ✓ | Difficult |

---

**Keep this handy for SLO implementation!**
