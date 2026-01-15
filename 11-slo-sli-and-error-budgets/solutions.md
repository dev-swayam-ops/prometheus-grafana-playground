# Solutions: SLO, SLI, and Error Budgets

## Solution 1: Define Service Level Indicator

**Choose SLI** (Example: Availability):
```promql
# API availability
up{job="api"}

# Returns: 1 (up) or 0 (down)
```

**Alternative SLI - Latency**:
```promql
# P95 response time
histogram_quantile(0.95, 
  rate(http_request_duration_seconds_bucket{job="api"}[5m])
)

# Returns: seconds (e.g., 0.45 = 450ms)
```

**Alternative SLI - Error Rate**:
```promql
# Error rate
sum(rate(http_requests_total{job="api",status=~"5.."}[5m])) /
sum(rate(http_requests_total{job="api"}[5m]))

# Returns: 0.0 to 1.0 (0.001 = 0.1%)
```

**Explanation**: 
- SLI must be quantifiable and measurable
- Pick metric that matters to users
- Availability: service responding
- Latency: requests complete in time
- Error rate: correct responses

---

## Solution 2: Set Service Level Objective

**Recommended SLOs by Tier**:
- **99.0%**: Internal/non-critical services
- **99.5%**: Standard web application
- **99.9%**: Production API, payment processing
- **99.95%**: Critical infrastructure

**Example for API Service**:
```
SLI: Availability (up metric)
SLO: 99.9%
SLI: Latency (P95 < 500ms)
SLO: 99.0%
SLI: Error Rate (4xx/5xx)
SLO: 0.1%
```

**Explanation**: 
- SLO is target value for SLI
- Must be achievable (99% yes, 99.999% maybe not)
- Should reflect user expectations
- Different SLOs for different tiers

---

## Solution 3: Calculate Monthly Error Budget

**Formula**:
```
Error Budget = (1 - SLO) × Seconds in Period

99.0% SLO:
(1 - 0.99) × 2,592,000 = 0.01 × 2,592,000 = 25,920 seconds
= 432 minutes = 7.2 hours/month

99.5% SLO:
(1 - 0.995) × 2,592,000 = 0.005 × 2,592,000 = 12,960 seconds
= 216 minutes = 3.6 hours/month

99.9% SLO:
(1 - 0.999) × 2,592,000 = 0.001 × 2,592,000 = 2,592 seconds
= 43.2 minutes/month
```

**For Different Periods**:
- Day: (1 - SLO) × 86,400
- Week: (1 - SLO) × 604,800
- Year: (1 - SLO) × 31,536,000

**Explanation**: 
- Error budget = amount allowed to fail
- If 99.9% SLO, can have 43.2 min downtime/month
- Team uses this to plan releases, incident response

---

## Solution 4: Query Error Rate

**Command**:
```bash
curl 'http://localhost:9090/api/v1/query' \
  -d 'query=rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m])'
```

**Query 1: Errors/sec**:
```promql
rate(http_requests_total{status=~"5.."}[5m])
```

**Output**:
```
0.0005  # 5 errors per 10,000 requests
```

**Query 2: Total requests/sec**:
```promql
rate(http_requests_total[5m])
```

**Output**:
```
10  # 10 requests per second
```

**Query 3: Error rate %**:
```promql
rate(http_requests_total{status=~"5.."}[5m]) / 
rate(http_requests_total[5m])
```

**Output**:
```
0.00005  # 0.005% error rate
```

**Explanation**: 
- errors/sec ÷ total_requests/sec = error_rate
- Multiply by 100 for percentage
- Use 5m window for stability

---

## Solution 5: Calculate Burn Rate

**Setup Example**:
```
SLO: 99.9% availability
Allowed error rate: 1 - 0.999 = 0.001 (0.1%)
```

**Query**:
```promql
# Current error rate
(1 - up{job="api"}) / (1 - 0.999)
```

**Interpretation**:
- Result = 1.0: Burning budget as expected (0.1% down)
- Result = 2.0: Burning budget at 2x rate (0.2% down)
- Result = 10.0: Burning budget at 10x rate (1% down)

**Example Values**:
```
If service 100% down:
(1 - 0) / 0.001 = 1000x burn rate (critical!)

If service 99% available:
(1 - 0.99) / 0.001 = 100x burn rate (urgent!)

If service 99.9% available:
(1 - 0.999) / 0.001 = 1x burn rate (expected)
```

**Explanation**: 
- Burn rate = actual error / budget error
- Tells you if error budget will run out
- 10x = 1 hour at this rate = breach in 30d

---

## Solution 6: Create High Burn Rate Alert

**Alert Rule**:
```yaml
- alert: SLOHighBurnRate
  expr: |
    (1 - avg_over_time(up{job="api"}[1h])) / 
    (1 - 0.999) > 10
  for: 5m
  labels:
    severity: critical
  annotations:
    summary: "SLO burning at 10x rate"
    description: "API at {{ $value }}x burn rate, SLO at risk"
```

**How it works**:
- Window: 1 hour (short term)
- Threshold: 10x burn rate
- For: 5 minutes (confirm issue)
- Severity: Critical

**Testing**:
```bash
# Simulate 10% error rate (0.001 is allowed)
# 10x would be 0.01 error rate
# (1 - 0.99) / 0.001 = 0.01 / 0.001 = 10

# Alert should fire immediately
```

**Explanation**: 
- High burn rate = imminent SLO breach
- 1 hour window detects acute issues
- Short "for" duration = quick alerting

---

## Solution 7: Create Medium Burn Rate Alert

**Alert Rule**:
```yaml
- alert: SLOMediumBurnRate
  expr: |
    (1 - avg_over_time(up{job="api"}[6h])) / 
    (1 - 0.999) > 2
  for: 30m
  labels:
    severity: warning
  annotations:
    summary: "SLO burning at 2x rate"
    description: "Sustained issues for 6h, {{ $value }}x burn"
```

**How it works**:
- Window: 6 hours (medium term)
- Threshold: 2x burn rate
- For: 30 minutes (confirm sustained)
- Severity: Warning

**Interpretation**:
- At 2x rate, budget runs out in 15 days
- Indicates systemic issue, not spike
- Requires investigation and fix

---

## Solution 8: Create Recording Rule

**Add to prometheus.yml**:
```yaml
groups:
  - name: slo_metrics
    interval: 30s
    rules:
      # Error budget consumed (30-day rolling window)
      - record: job:slo:error_budget:consumed
        expr: 1 - avg_over_time(up{job="api"}[30d])
        labels:
          slo: "99.9"
      
      # Burn rate (current error rate / allowed)
      - record: job:slo:burn_rate
        expr: |
          (1 - avg_over_time(up{job="api"}[1h])) / 
          (1 - 0.999)
```

**Verify**:
```bash
curl http://localhost:9090/rules | grep slo
```

**Explanation**: 
- Recording rules pre-compute expensive queries
- Store as metrics for later use
- Rolling window tracks budget over time

---

## Solution 9: Query Error Budget Remaining

**Query**:
```promql
# Budget remaining as percentage
(
  (1 - 0.999) - (1 - avg_over_time(up{job="api"}[30d]))
) / (1 - 0.999) * 100
```

**Breakdown**:
```
Total budget: 1 - 0.999 = 0.001 (0.1%)
Consumed: 1 - avg_availability
Remaining: Total - Consumed
Percentage: (Remaining / Total) × 100
```

**Examples**:
```
If 99.95% available over 30d:
Consumed = 1 - 0.9995 = 0.0005
Remaining = 0.001 - 0.0005 = 0.0005
Percentage = (0.0005 / 0.001) × 100 = 50% remaining

If 99.85% available:
Consumed = 1 - 0.9985 = 0.0015
Remaining = 0.001 - 0.0015 = -0.0005 (negative! breached)
```

---

## Solution 10: Create SLO Dashboard

**Grafana Panel Steps**:
1. Create new dashboard
2. Add panel: Stat
3. Query:
```promql
avg_over_time(up{job="api"}[30d]) * 100
```
4. Thresholds: Red < 99.9%, Yellow < 99.95%, Green >= 99.95%
5. Unit: Percent
6. Display: "API Availability 30d"

**Add Second Panel - Error Budget**:
```promql
(
  (1 - 0.999) - (1 - avg_over_time(up{job="api"}[30d]))
) / (1 - 0.999) * 100
```

**Third Panel - Burn Rate**:
```promql
(1 - avg_over_time(up{job="api"}[1h])) / (1 - 0.999)
```

**Explanation**: 
- Status shows if SLO met (green/red)
- Budget panel shows consumption
- Burn rate panel shows trend
- Updates every 5 minutes

---

**All Solutions Complete ✅**
