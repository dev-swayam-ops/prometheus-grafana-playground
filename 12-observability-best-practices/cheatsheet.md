# Cheatsheet: Observability Best Practices

## Metric Naming Convention

**Pattern**: `<namespace>_<subsystem>_<name>_<unit>`

| Part | Meaning | Example |
|------|---------|---------|
| namespace | Project/domain | http, database, cache |
| subsystem | Component | request, connection, hit |
| name | What measured | duration, count, size |
| unit | Measurement | seconds, bytes, total |

**Full Example**:
```
http_request_duration_seconds
├─ http (namespace)
├─ request (subsystem)
├─ duration (name)
└─ seconds (unit)
```

## Metric Types and Conventions

| Type | Convention | Example |
|------|-----------|---------|
| **Counter** | Ends with `_total` | http_requests_total |
| **Gauge** | Descriptive name | node_memory_free_bytes |
| **Histogram** | `_bucket`, `_sum`, `_count` | http_duration_seconds_bucket |
| **Summary** | `_count`, `_sum` | query_duration_seconds_sum |

## Label Guidelines

| Guideline | Good | Bad |
|-----------|------|-----|
| **Count** | < 10 labels | > 20 labels |
| **Cardinality** | < 1K combinations | 1M combinations |
| **Names** | job, instance, status | user_id, session_id |
| **Values** | "production", "api" | "user-123-xyz" |
| **Bounded** | enum values | timestamps, IDs |

**Label Cardinality Formula**:
```
Total Series = Label1_Values × Label2_Values × ... × LabelN_Values

Bad: user_id (1M) × request_id (10M) = 10 trillion series!
Good: job (10) × status (5) = 50 series
```

## Cardinality Commands

| Query | Purpose |
|-------|---------|
| `count(count by (__name__)(up))` | Total time series |
| `topk(10, count by (__name__)(up))` | Top 10 metrics |
| `count by (job)(http_requests_total)` | Cardinality per label |

## Common Units and Suffixes

| Unit | Suffix | Example |
|------|--------|---------|
| Seconds | `_seconds` | response_time_seconds |
| Bytes | `_bytes` | memory_usage_bytes |
| Count | `_total` | errors_total |
| Ratio | none | cpu_utilization (0-1) |
| Percentage | `_percent` | disk_usage_percent |
| Rate | `_per_second` | requests_per_second |

## Alert Best Practices

| Practice | Example | Benefit |
|----------|---------|---------|
| **Meaningful name** | HTTPErrorRateHigh | Clear what alert is |
| **Clear threshold** | > 5% | Quantified condition |
| **For duration** | for: 5m | Avoid false positives |
| **Runbook** | Add link | Faster resolution |
| **Severity** | critical/warning | Prioritization |

**Alert Template**:
```yaml
- alert: DescriptiveAlertName
  expr: metric > threshold
  for: 5m
  labels:
    severity: critical
  annotations:
    summary: "Description"
    runbook: "https://wiki.com/runbook"
```

## Observability Pillars

| Pillar | Tool | Purpose |
|--------|------|---------|
| **Metrics** | Prometheus | Numbers over time |
| **Logs** | ELK/Loki | Events and details |
| **Traces** | Jaeger | Request flow |
| **Together** | Correlation IDs | Root cause analysis |

## Documentation Template

```markdown
# Metric: http_requests_total

**Type**: Counter
**Unit**: Requests
**Source**: Application
**Labels**: job, status, method

**Description**:
Total HTTP requests processed, counted by status code and method.

**Example Query**:
rate(http_requests_total[5m])

**Alert Threshold**:
> 1000 errors/sec for 5m = critical

**Investigation**:
1. Check error logs
2. Review slow queries
3. Check downstream services
```

## Recording Rules for Performance

```yaml
groups:
  - name: common_queries
    interval: 30s
    rules:
      # Pre-compute frequently used query
      - record: http:request_rate:5m
        expr: sum(rate(http_requests_total[5m])) by (job)
      
      # Error rate calculation
      - record: http:error_rate:5m
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m])) /
          sum(rate(http_requests_total[5m]))
```

## Cardinality Reduction Strategies

| Issue | Solution | Effort |
|-------|----------|--------|
| Too many status codes | Only track 2xx, 4xx, 5xx | Low |
| Per-user tracking | Track by service only | Medium |
| Request IDs as labels | Use sampling instead | Medium |
| Timestamps | Use time-series instead | High |

## Performance Checklist

✅ **DO**:
- Use consistent naming across metrics
- Add descriptive labels (< 10)
- Document what metrics mean
- Create runbooks for alerts
- Monitor cardinality growth
- Use recording rules for common queries
- Review metrics quarterly

❌ **DON'T**:
- Use user_id, session_id, request_id as labels
- Add timestamps to metric labels
- Alert on every metric change
- Name metrics randomly
- Forget to document metrics
- Use high-frequency sampled data as metric
- Set alerts without testing threshold

## Correlation IDs for Logs + Traces

```
Request Flow:
API → trace_id=abc123
      ├─ Log: trace_id=abc123
      └─ Database query: trace_id=abc123

Benefits:
- Follow request through system
- Correlate metrics/logs/traces
- Find root cause quickly
```

## Sample Runbook Structure

```markdown
# Alert: {{ alert_name }}

## Detection
- **Metric**: {{ metric_name }}
- **Threshold**: {{ threshold }}
- **Window**: {{ duration }}

## Impact
- User facing? Yes/No
- Service degraded? Yes/No
- Data loss risk? Yes/No

## Diagnosis
1. SSH to server
2. Run diagnostic command
3. Check logs

## Recovery
- Step 1: ...
- Step 2: ...
- Step 3: ...

## Prevention
- Increase capacity
- Optimize query
- Review logs for pattern
```

---

**Keep this handy for monitoring design!**
