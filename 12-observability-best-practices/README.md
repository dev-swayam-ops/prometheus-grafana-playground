# Observability Best Practices

## What You'll Learn
- Observability pillars: metrics, logs, traces
- Naming conventions for metrics
- Cardinality management and performance
- Alert design and tuning
- Documentation and runbooks
- Cost optimization in monitoring

## Prerequisites
- Module 01: Prometheus Basics
- Module 03: PromQL Queries
- Module 04: Recording Rules and Alerting
- Module 06: Grafana Basics

## Key Concepts

### Observability Pillars
1. **Metrics** - Numbers over time (Prometheus)
2. **Logs** - Events and details (ELK, Loki)
3. **Traces** - Request flow (Jaeger, Zipkin)
4. **Together** - Correlate to understand issues

### Metrics Naming Convention
```
<namespace>_<subsystem>_<name>_<unit>

Examples:
- http_requests_total (counter, total requests)
- http_request_duration_seconds (histogram, latency)
- node_memory_free_bytes (gauge, available memory)
- database_connections_active (gauge, current connections)
```

### Label Guidelines
- **DO**: Add labels that help group/filter (job, instance, service)
- **DON'T**: Add high-cardinality labels (user_id, request_id, timestamps)
- **Limit**: < 10 labels per metric
- **Example**: `http_requests_total{job="api", status="200", method="GET"}`

### High Cardinality Problems
```
Bad: http_requests{user_id="123", request_id="abc"}
     = 1M users × 1M requests = 1 trillion time series!

Good: http_requests_total{service="api", status="200"}
      = few thousand time series
```

## Hands-on Lab

**Objective**: Apply observability best practices to monitoring setup.

### Step 1: Audit Current Metrics
```bash
# Query all metric names
curl 'http://localhost:9090/api/v1/label/__name__/values' | jq '.data | length'

# Check metric cardinality
curl 'http://localhost:9090/api/v1/query?query=count(count%20by%20(__name__)%20(up))'
```

### Step 2: Create Naming Convention Document
```
Metrics follow: <namespace>_<subsystem>_<name>_<unit>

Example: http_request_duration_seconds
- Namespace: http
- Subsystem: request
- Name: duration
- Unit: seconds

All counters end with _total
All histograms end with _bucket, _count, _sum
```

### Step 3: Implement Label Best Practices
Good metrics:
```yaml
# Good: Bounded labels
http_requests_total{job="api", status="200", method="GET"} 100

# Good: Service identification
mysql_connections_active{service="payments", instance="host1"}
```

Bad metrics (remove):
```yaml
# Bad: High cardinality
api_request{user_id="123", session_id="xyz", ...}  
# Avoids: 1M+ unique combinations
```

### Step 4: Set Up Cardinality Alerts
```yaml
groups:
  - name: cardinality_alerts
    rules:
      - alert: HighMetricCardinality
        expr: count(count by (__name__)(up)) > 100000
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Too many time series ({{ $value }})"
```

### Step 5: Create Alert Runbook
```markdown
# High CPU Alert Runbook

## Symptom
Alert: node_cpu_seconds_total > 80%

## Investigation
1. ssh into server: `ssh user@{{ $labels.instance }}`
2. Check top processes: `top -b -n 1 | head -20`
3. Check disk I/O: `iostat -x 1 5`
4. Review logs: `journalctl -xe --tail=100`

## Resolution
- Kill runaway process: `kill -9 PID`
- Restart service: `systemctl restart myservice`
- Scale up if under load

## Prevention
- Set resource limits in docker
- Monitor trends, not just current
```

### Step 6: Document Metrics
Create metrics.md:
```markdown
# Metrics Dictionary

## HTTP Metrics
- `http_requests_total`: Total HTTP requests by status/method
- `http_request_duration_seconds`: Request latency (histogram)
- `http_requests_in_flight`: Current active requests

## Database Metrics
- `db_connections_active`: Active connections
- `db_query_duration_seconds`: Query execution time
- `db_pool_exhausted_total`: Times pool ran out
```

## Validation
✅ Metrics follow naming convention
✅ Labels have low cardinality (< 10K unique combinations per metric)
✅ Runbooks exist for critical alerts
✅ Alert rules documented with examples
✅ Metrics indexed efficiently

## Cleanup
```bash
# Review and remove high-cardinality metrics
# Clean up unused recordings rules
# Archive old dashboards
```

## Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| High cardinality labels | Prometheus OOM | Limit unique label values |
| Inconsistent naming | Hard to find metrics | Use naming convention |
| No alert runbooks | Slow incident response | Document each alert |
| Over-instrumentation | Performance issues | Measure what matters to users |
| Alerts without SLOs | Alert fatigue | Define when to alert based on SLO |

## Troubleshooting

**Prometheus using too much memory**:
```bash
# Find high-cardinality metrics
curl 'http://localhost:9090/api/v1/query?query=topk(10, count by (__name__)(up))'

# Check label combinations
curl 'http://localhost:9090/api/v1/query?query=http_requests_total' | jq '.data.result | length'
```

**Alerts firing too frequently**:
- Check alert thresholds are reasonable
- Increase "for" duration to filter spikes
- Verify metric quality/stability

**Dashboards slow to load**:
- Reduce time range
- Simplify queries (use recording rules)
- Check browser console for errors

## Next Steps
- Module 13: Troubleshooting and Debugging
- Module 14: Real-world Monitoring Labs
- Advanced: Distributed tracing, log correlation
