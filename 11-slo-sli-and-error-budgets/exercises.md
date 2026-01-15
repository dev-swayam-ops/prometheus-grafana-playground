# Exercises: SLO, SLI, and Error Budgets

## Exercise 1: Define Service Level Indicator (Easy)
Identify measurable SLI for your service.

**Task**:
Choose one of:
1. Availability: `up{job="api"}`
2. Latency: `histogram_quantile(0.95, ...)`
3. Error rate: `errors / requests`

Write PromQL query for your chosen SLI.

**Validation**: Query returns numeric values in Prometheus.

---

## Exercise 2: Set Service Level Objective (Easy)
Define target value for SLI.

**Task**:
Pick realistic SLO:
- 99% availability
- 99.5% availability  
- 99.9% availability

Document: "Our service will be available {{ SLO% }} of the time."

**Validation**: SLO documented and agreed with team.

---

## Exercise 3: Calculate Monthly Error Budget (Easy)
Determine allowed downtime.

**Task**:
Formula: (1 - SLO%) × seconds_in_month
```
99.0% SLO = 0.01 × 2,592,000s = 25,920s = 432 minutes
99.5% SLO = 0.005 × 2,592,000s = 12,960s = 216 minutes
99.9% SLO = 0.001 × 2,592,000s = 2,592s = 43.2 minutes
```

Calculate for your SLO.

**Validation**: Error budget minutes/hours calculated correctly.

---

## Exercise 4: Query Error Rate (Easy)
Calculate percentage of failed requests.

**Task**:
```promql
# Failed requests per second
rate(http_requests_total{status=~"5.."}[5m])

# Total requests per second
rate(http_requests_total[5m])

# Error rate percentage
rate(http_requests_total{status=~"5.."}[5m]) / 
rate(http_requests_total[5m])
```

Run all three queries in Prometheus.

**Validation**: Queries return valid numbers (0.0 to 1.0 for rate).

---

## Exercise 5: Calculate Burn Rate (Medium)
Determine how fast error budget is consumed.

**Task**:
```promql
# Current error rate
current_error_rate = 0.001

# Allowed error rate for SLO
allowed_error_rate = 1 - 0.999 = 0.001

# Burn rate = current / allowed
burn_rate = current_error_rate / allowed_error_rate
# Result: 1.0 = consuming budget as expected
```

Create query to compute burn rate.

**Validation**: Burn rate returns 1.0 when error rate = (1-SLO).

---

## Exercise 6: Create Burn Rate Alert - High (Medium)
Alert when burning budget at 10x rate.

**Task**:
```promql
# High burn (10x) should trigger
rate(errors[1h]) / (1 - 0.999) > 10

# Alert condition
(rate(http_requests_total{status=~"5.."}[1h]) / 
 rate(http_requests_total[1h])) > 10 * 0.001
```

Add to prometheus alert rules.

**Validation**: Alert fires when error rate > 0.01.

---

## Exercise 7: Create Burn Rate Alert - Medium (Medium)
Alert when burning budget at 2x rate (6-hour window).

**Task**:
```promql
(rate(http_requests_total{status=~"5.."}[6h]) / 
 rate(http_requests_total[6h])) > 2 * (1 - 0.999)
```

Add alert for sustained issues.

**Validation**: Alert fires for prolonged errors.

---

## Exercise 8: Create Recording Rule for Error Budget (Medium)
Pre-compute error budget consumption.

**Task**:
```yaml
- record: job:error_budget:consumed
  expr: 1 - avg_over_time(up{job="api"}[30d])
```

Verify rule evaluates without errors.

**Validation**: Recording rule appears in /rules endpoint.

---

## Exercise 9: Query Error Budget Remaining (Medium)
Calculate how much budget is left in period.

**Task**:
```promql
# Budget consumed so far (30 days)
consumed = 1 - avg(up[30d])

# Remaining budget
remaining = (1 - 0.999) - consumed
```

Write query to show remaining budget percentage.

**Validation**: Returns value between 0-100%.

---

## Exercise 10: Create SLO Dashboard Panel (Medium)
Display SLO status in Grafana.

**Task**:
Create panel with:
1. Title: "API SLO Status"
2. Query: Availability over last 30 days
3. Threshold: 99.9%
4. Show: % remaining in error budget

**Validation**: Panel displays SLO target vs actual availability.

---

**All Exercises Complete ✅**
