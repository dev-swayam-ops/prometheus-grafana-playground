# Exercises: Observability Best Practices

## Exercise 1: Review Metric Names (Easy)
Audit existing metrics for naming convention.

**Task**:
1. Query all metrics: `curl 'http://localhost:9090/api/v1/label/__name__/values'`
2. Check 10 metrics for: `namespace_subsystem_name_unit` pattern
3. List any metrics that don't follow pattern

**Validation**: Identify 3+ metrics needing rename.

---

## Exercise 2: Identify High-Cardinality Labels (Easy)
Find metrics with too many unique label combinations.

**Task**:
```promql
# Count unique combinations per metric
count by (__name__)(count by (__name__, job, instance)(up))

# Find top 5 highest cardinality
topk(5, count by (__name__)(up))
```

Run queries in Prometheus.

**Validation**: Can list top metrics by cardinality.

---

## Exercise 3: Check Label Count (Easy)
Verify each metric has < 10 labels.

**Task**:
1. Pick a metric: `http_requests_total`
2. Query: `http_requests_total` in Prometheus
3. Count unique labels in results
4. Verify < 10 labels

**Validation**: Label count documented for 3 metrics.

---

## Exercise 4: Create Metric Naming Guide (Easy)
Document naming convention.

**Task**:
Create document with:
```
Format: <namespace>_<subsystem>_<name>_<unit>

Examples:
- http_request_duration_seconds
- database_connections_active
- redis_memory_used_bytes
```

**Validation**: Guide written with 5+ examples.

---

## Exercise 5: Remove High-Cardinality Label (Medium)
Edit exporter to remove problematic label.

**Task**:
Find metric with high cardinality:
```promql
# Bad: user_id label (millions unique)
api_calls{user_id="123", service="api"}

# Fix: Remove user_id, keep service
api_calls{service="api"}
```

Document change and rationale.

**Validation**: Change recorded with before/after.

---

## Exercise 6: Create Alert Runbook (Medium)
Write runbook for critical alert.

**Task**:
For alert: `HighMemoryUsage > 85%`

Write runbook with:
1. Symptom
2. Investigation steps
3. Resolution steps
4. Prevention tips

**Validation**: Runbook has all 4 sections.

---

## Exercise 7: Set Cardinality Alert (Medium)
Create alert for too many time series.

**Task**:
```yaml
- alert: HighMetricCardinality
  expr: count(count by (__name__)(up)) > 100000
  for: 5m
```

Add to prometheus alert rules.

**Validation**: Alert created, tests firing condition.

---

## Exercise 8: Document Metrics Dictionary (Medium)
Create reference for all important metrics.

**Task**:
List 5 key metrics with:
- Name
- Type (counter, gauge, histogram)
- Purpose
- Example labels

Format as markdown table.

**Validation**: Table has 5+ metrics documented.

---

## Exercise 9: Create Query Guidelines (Medium)
Document recommended queries for common tasks.

**Task**:
Create guide with queries for:
- Current CPU usage
- Request error rate
- Database connection pool status
- Memory trending

**Validation**: 4+ queries documented with explanations.

---

## Exercise 10: Audit Label Cardinality (Medium)
Run full cardinality audit on all metrics.

**Task**:
```promql
# Get all metrics
topk(20, count by (__name__)(up))

# For each top metric, count combinations
count by (job)(http_requests_total)
```

Document findings and recommendations.

**Validation**: Report with top 10 metrics by cardinality.

---

**All Exercises Complete ✅**
