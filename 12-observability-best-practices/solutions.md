# Solutions: Observability Best Practices

## Solution 1: Review Metric Names

**Steps**:
```bash
# Get all metric names
curl 'http://localhost:9090/api/v1/label/__name__/values' | jq '.data | sort'
```

**Expected Output**:
```json
[
  "go_gc_duration_seconds",
  "go_goroutines",
  "http_request_duration_seconds",
  "http_requests_total",
  "node_cpu_seconds_total",
  "node_memory_free_bytes",
  "prometheus_http_request_duration_seconds",
  "up"
]
```

**Check Pattern**: `<namespace>_<subsystem>_<name>_<unit>`
- ✓ `http_request_duration_seconds` (http_request_duration_seconds)
- ✓ `node_cpu_seconds_total` (node_cpu_seconds_total)
- ✗ `custom_metric` (missing subsystem/unit)

**Explanation**: Consistent naming helps organization and discoverability.

---

## Solution 2: Identify High-Cardinality Labels

**Query 1: Count Unique Values**:
```promql
count by (__name__)(up)
```

**Output**:
```
http_requests_total: 10,000
node_network_transmit_bytes: 50,000
mysql_query_duration_seconds: 5,000
```

**Query 2: Top High-Cardinality Metrics**:
```promql
topk(5, count by (__name__)(up))
```

**Check**: Metrics > 100K unique combinations are problematic.

**Explanation**: High cardinality = many time series = high memory/CPU.

---

## Solution 3: Check Label Count

**Command**:
```bash
# Pick metric and query
curl 'http://localhost:9090/api/v1/query?query=http_requests_total' | jq '.data.result[0].metric | keys | length'
```

**Expected Output**:
```
5  # 5 labels is good
```

**Labels Found**:
```json
{
  "job": "api",
  "instance": "localhost:9100",
  "status": "200",
  "method": "GET",
  "endpoint": "/api"
}
```

**Count**: 5 labels ✓ (< 10 limit)

**Explanation**: Each label adds cardinality, 10 is practical limit.

---

## Solution 4: Create Metric Naming Guide

**Format Document**:
```markdown
# Metric Naming Convention

Pattern: <namespace>_<subsystem>_<name>_<unit>

## Rules
1. All lowercase, underscores separate words
2. Counters end with _total
3. Gauges have descriptive unit (bytes, seconds, etc.)
4. Histograms generate _bucket, _sum, _count

## Examples
- http_request_duration_seconds (histogram)
- http_requests_total (counter)
- node_memory_free_bytes (gauge)
- database_connections_active (gauge)
- cache_hits_total (counter)
- api_latency_seconds (histogram)
```

**Explanation**: Convention improves readability and searchability.

---

## Solution 5: Remove High-Cardinality Label

**Original (Bad)**:
```yaml
- name: api_calls
  help: API calls by user
  type: counter
  labels: [user_id, service, status]  # user_id = 1M unique values!
```

**Fixed (Good)**:
```yaml
- name: api_calls_total
  help: API calls by service
  type: counter
  labels: [service, status]  # Only 10 unique combinations
```

**Change Record**:
```
Metric: api_calls_total
Removed: user_id label (1M+ cardinality)
Reason: Caused prometheus memory spike to 8GB
Kept: service, status (low cardinality)
Impact: Reduced from 1M series to 50 series
```

**Explanation**: Remove personal data or high-cardinality fields.

---

## Solution 6: Create Alert Runbook

**Template**:
```markdown
# Alert Runbook: HighMemoryUsage

## Alert Name
`node_memory_available_percent < 15%`

## Symptom
- Memory available drops below 15%
- Prometheus alert fires
- Likely slow response or OOM

## Investigation Steps
1. SSH to affected host
   `ssh user@{{ $labels.instance }}`

2. Check memory usage
   `free -h`
   `ps aux --sort=-%mem | head -20`

3. Check disk swap
   `swapon --show`

4. Review system logs
   `journalctl -e --tail=50`

## Resolution
- Kill memory hog: `kill -9 PID`
- Restart service: `systemctl restart service`
- Clear cache: `sync; echo 3 > /proc/sys/vm/drop_caches`

## Prevention
- Set memory limits: `--memory=2g`
- Monitor trends weekly
- Plan capacity before hitting limits
```

**Explanation**: Runbooks reduce MTTR (mean time to resolution).

---

## Solution 7: Set Cardinality Alert

**Prometheus Alert Rule**:
```yaml
groups:
  - name: cardinality
    interval: 5m
    rules:
      - alert: HighMetricCardinality
        expr: |
          count(count by (__name__)(up)) > 100000
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Prometheus has {{ $value }} time series"
          description: "Reduce cardinality or increase memory"
```

**How it Works**:
```
count by (__name__)(up) = count unique for each metric
count(...) = total unique time series
> 100000 = threshold (adjust for your setup)
for: 5m = wait 5 min to confirm
```

**Testing**:
```bash
# Check if alert would fire
curl 'http://localhost:9090/api/v1/query?query=count(count%20by%20(__name__)(up))'

# Should return number > 100000 to trigger
```

---

## Solution 8: Document Metrics Dictionary

**Markdown Table**:
```markdown
| Metric | Type | Purpose | Example Labels |
|--------|------|---------|-----------------|
| http_requests_total | Counter | Total HTTP requests | job, status, method |
| http_request_duration_seconds | Histogram | Request latency | job, endpoint |
| node_memory_free_bytes | Gauge | Available memory | instance, numa_node |
| database_connections_active | Gauge | Active connections | instance, database |
| cache_hits_total | Counter | Cache hit count | service, cache_type |
```

**Explanation**: Dictionary helps team find right metric for queries.

---

## Solution 9: Create Query Guidelines

**Document**:
```markdown
# Common Queries

## CPU Usage (Last 5 minutes)
\`\`\`promql
(1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m]))) * 100
\`\`\`

## Request Error Rate
\`\`\`promql
sum(rate(http_requests_total{status=~"5.."}[5m])) /
sum(rate(http_requests_total[5m])) * 100
\`\`\`

## Database Connection Pool Status
\`\`\`promql
database_connections_active / database_connections_max
\`\`\`

## Memory Trending (7-day)
\`\`\`promql
avg_over_time(node_memory_free_bytes[7d])
\`\`\`
```

---

## Solution 10: Audit Label Cardinality

**Commands**:
```bash
# Get top metrics by cardinality
curl 'http://localhost:9090/api/v1/query?query=topk(10,%20count%20by%20(__name__)(up))' | jq

# Drill into specific metric
curl 'http://localhost:9090/api/v1/query?query=count%20by%20(job,instance)(http_requests_total)' | jq
```

**Report Template**:
```markdown
# Cardinality Audit Report

## Top Metrics by Cardinality
| Metric | Series Count | Status |
|--------|--------------|--------|
| http_requests_total | 15,000 | ⚠️ High |
| node_network_transmit_bytes | 8,000 | ✓ OK |
| mysql_connections | 500 | ✓ OK |
| custom_events | 250,000 | 🔴 CRITICAL |

## Recommendations
1. Reduce custom_events labels
2. Monitor http_requests_total growth
3. Set cardinality alert at 100K

## Actions Taken
- Removed user_id from custom_events
- Added cardinality alert
- Documented in metrics guide
```

---

**All Solutions Complete ✅**
