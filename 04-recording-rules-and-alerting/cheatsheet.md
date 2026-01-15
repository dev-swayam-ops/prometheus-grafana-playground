# Cheatsheet: Recording Rules and Alerting

## Rule File Structure

```yaml
groups:
  - name: group_name
    interval: 15s  # Evaluation interval
    rules:
      # Recording rule
      - record: new_metric_name
        expr: promql_expression

      # Alert rule
      - alert: AlertName
        expr: condition_expression
        for: 5m  # Duration before firing
        labels:
          key: value
        annotations:
          description: "Text with {{ $labels.instance }}"
```

## Recording Rules

| Field | Purpose | Example |
|-------|---------|---------|
| `record` | New metric name | `job:requests:rate5m` |
| `expr` | PromQL to evaluate | `rate(requests_total[5m])` |
| `interval` | Evaluation frequency | `15s` (default from global) |

## Naming Convention for Recording Rules

```
<level>:<metric>:<aggregation><period>
```

| Part | Example | Meaning |
|------|---------|---------|
| Level | `job`, `instance`, `node` | Aggregation dimension |
| Metric | `requests`, `memory`, `cpu` | What's measured |
| Aggregation | `rate`, `avg`, `sum` | How it's calculated |
| Period | `5m`, `1h` | Time window |

**Examples**:
- `job:http_requests:rate5m` - 5m request rate per job
- `instance:memory:percent_used` - Memory % per instance
- `node:cpu:avg1h` - 1h avg CPU per node

## Alert Rules

| Field | Purpose | Example |
|-------|---------|---------|
| `alert` | Alert name | `HighCPU` |
| `expr` | Condition PromQL | `cpu_usage > 0.8` |
| `for` | Duration before firing | `5m`, `10m`, `1h` |
| `labels` | Custom metadata | `severity: critical` |
| `annotations` | Description text | `summary: "{{ $labels.instance }}"` |

## Alert States

| State | Meaning | Duration |
|-------|---------|----------|
| **Inactive** | Condition false | Condition not met |
| **Pending** | Condition true | Until "for" duration reached |
| **Firing** | Alert active | Condition true + "for" duration met |

## Common Aggregation Functions for Rules

| Function | Purpose | Example |
|----------|---------|---------|
| `sum()` | Total | `sum(rate(metric[5m]))` |
| `avg()` | Average | `avg(metric) by (job)` |
| `min()` | Minimum | `min(metric) by (instance)` |
| `max()` | Maximum | `max(metric) by (instance)` |
| `rate()` | Per-second rate | `rate(counter[5m])` |
| `increase()` | Total change | `increase(counter[1h])` |

## Template Variables in Annotations

| Variable | Example Value | Use |
|----------|---------------|-----|
| `{{ $labels.instance }}` | `localhost:9100` | Reference label |
| `{{ $labels.severity }}` | `critical` | Reference any label |
| `{{ $value }}` | `85.5` | Metric value |
| `{{ $timestamp }}` | `1705329600` | Unix timestamp |

## Alert Annotation Examples

```yaml
annotations:
  summary: "High CPU on {{ $labels.instance }}"
  description: "CPU usage is {{ $value | humanizePercentage }}"
  runbook: "https://example.com/runbooks/high-cpu"
  dashboard: "https://grafana.example.com/d/cpu"
```

## Prometheus Configuration for Rules

```yaml
# Minimal config
global:
  evaluation_interval: 15s

rule_files:
  - /etc/prometheus/rules.yml

alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - alertmanager:9093

scrape_configs:
  # ... scrape configs ...
```

## Useful Prometheus API Endpoints

| Endpoint | Purpose |
|----------|---------|
| `/rules` | View all rules (browser) |
| `/alerts` | View alert status (browser) |
| `/api/v1/rules` | API: List all rules |
| `/api/v1/alerts` | API: List alert status |
| `/api/v1/query?query=ALERTS` | API: Query ALERTS metric |

## curl Commands for Rules

| Command | Purpose |
|---------|---------|
| `curl http://localhost:9090/api/v1/rules` | Get all rules |
| `curl http://localhost:9090/api/v1/alerts` | Get alert status |
| `curl -G 'http://localhost:9090/api/v1/query' --data-urlencode 'query=ALERTS'` | Query ALERTS metric |
| `curl http://localhost:9090/api/v1/targets` | Get scrape targets |

## Common Alert Expressions

| Scenario | Expression |
|----------|-----------|
| **High CPU** | `(1 - rate(idle[5m])) > 0.8` |
| **Memory low** | `MemAvailable / MemTotal < 0.1` |
| **Target down** | `up == 0` |
| **High error rate** | `rate(errors[5m]) / rate(requests[5m]) > 0.05` |
| **Disk full** | `disk_used / disk_total > 0.9` |
| **Queue building** | `increase(queue_size[5m]) > 0` |
| **Slow response** | `histogram_quantile(0.95, latency) > 1` |

## Rule Validation

**Check YAML syntax**:
```bash
yamllint rules.yml
```

**Validate with Prometheus**:
```bash
docker run --rm -v $(pwd)/rules.yml:/rules.yml \
  prom/prometheus --config.file=/dev/null --rules /rules.yml
```

**Dry-run**:
```bash
docker run --rm -it -v $(pwd):/prometheus \
  prom/prometheus --dry-run
```

## Debugging Tips

| Issue | Debug Command |
|-------|---------------|
| Rules not loading | `docker logs prometheus \| grep rule` |
| Alert not firing | `curl http://localhost:9090/alerts` |
| Expression error | Test in Graph tab first |
| Wrong interval | Check global + group interval |
| No alert state | Verify "for" duration reached |

## Recording Rule Best Practices

✅ **DO**:
- Name rules consistently (use convention)
- Use appropriate aggregations
- Document complex expressions
- Test expressions before recording

❌ **DON'T**:
- Create too many recording rules (maintenance burden)
- Use very short intervals (creates too many points)
- Record without clear purpose
- Forget to remove old rules

---

**Keep this handy while creating rules and alerts!**
