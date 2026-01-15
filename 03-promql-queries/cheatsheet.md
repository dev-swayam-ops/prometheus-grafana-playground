# Cheatsheet: PromQL Queries

## Query Types

| Type | Syntax | Returns | Example |
|------|--------|---------|---------|
| Instant Vector | `metric` | Current values | `up` → 1, 1, 0 |
| Range Vector | `metric[duration]` | Historical values | `up[5m]` → values over 5m |
| Scalar | Number | Single value | `42` or `3.14` |

## Time Duration Syntax

| Duration | Meaning | Example |
|----------|---------|---------|
| `s` | seconds | `30s`, `60s` |
| `m` | minutes | `5m`, `10m` |
| `h` | hours | `1h`, `24h` |
| `d` | days | `7d`, `30d` |
| `w` | weeks | `4w` |
| `y` | years | `1y` |

## Operators

### Comparison Operators

| Op | Meaning | Example |
|----|---------|---------|
| `==` | Equal | `up == 1` |
| `!=` | Not equal | `up != 0` |
| `>` | Greater than | `errors > 100` |
| `<` | Less than | `errors < 50` |
| `>=` | Greater or equal | `cpu >= 80` |
| `<=` | Less or equal | `mem <= 90` |

### Arithmetic Operators

| Op | Meaning | Example |
|----|---------|---------|
| `+` | Add | `a + b` |
| `-` | Subtract | `a - b` |
| `*` | Multiply | `a * 100` |
| `/` | Divide | `a / b` |
| `%` | Modulo | `a % 10` |
| `^` | Power | `a ^ 2` |

## Label Matching

| Matcher | Meaning | Example |
|---------|---------|---------|
| `=` | Exact match | `job="prometheus"` |
| `!=` | Not equal | `job!="backup"` |
| `=~` | Regex match | `job=~"prom.*"` |
| `!~` | Not regex match | `job!~"backup.*"` |

## Aggregation Functions

| Function | Purpose | Example |
|----------|---------|---------|
| `sum()` | Total | `sum(rate(...))` |
| `avg()` | Average | `avg(cpu)` |
| `min()` | Minimum | `min(latency)` |
| `max()` | Maximum | `max(memory)` |
| `count()` | Count series | `count(up)` |
| `count_values()` | Count by value | `count_values("status", up)` |
| `topk()` | Top K values | `topk(3, memory)` |
| `bottomk()` | Bottom K values | `bottomk(3, memory)` |

## Range Vector Functions

| Function | Purpose | Example |
|----------|---------|---------|
| `rate()` | Per-second change | `rate(counter[5m])` |
| `increase()` | Total increase | `increase(counter[1h])` |
| `delta()` | Change value | `delta(gauge[5m])` |
| `irate()` | Instant rate | `irate(counter[5m])` |
| `avg_over_time()` | Average over time | `avg_over_time(gauge[1h])` |
| `min_over_time()` | Min over time | `min_over_time(gauge[1h])` |
| `max_over_time()` | Max over time | `max_over_time(gauge[1h])` |
| `sum_over_time()` | Sum over time | `sum_over_time(gauge[1h])` |

## Other Functions

| Function | Purpose | Example |
|----------|---------|---------|
| `abs()` | Absolute value | `abs(change)` |
| `ceil()` | Round up | `ceil(3.14)` |
| `floor()` | Round down | `floor(3.14)` |
| `round()` | Round | `round(3.5)` |
| `sqrt()` | Square root | `sqrt(value)` |
| `log()` | Natural log | `log(value)` |

## Common Query Patterns

| Use Case | Query |
|----------|-------|
| **Target health** | `up` or `count(up == 1)` |
| **Request rate** | `rate(requests_total[5m])` |
| **Error rate** | `rate(errors_total[5m]) / rate(requests_total[5m])` |
| **CPU usage %** | `(1 - avg(rate(idle[5m]))) * 100` |
| **Memory usage %** | `(1 - MemAvailable/MemTotal) * 100` |
| **Top 5 consumers** | `topk(5, process_memory_usage)` |
| **P95 latency** | `histogram_quantile(0.95, latency)` |
| **5-minute average** | `avg_over_time(metric[5m])` |

## Label Filtering Examples

| Query | Result |
|-------|--------|
| `metric` | All series |
| `metric{job="node"}` | Only node job |
| `metric{job!="backup"}` | All except backup |
| `metric{mode=~"idle\|system"}` | Idle or system modes |
| `metric{instance=~"host[0-9]"}` | host0 through host9 |

## API Query Examples

**Instant Query**:
```bash
curl -G 'http://localhost:9090/api/v1/query' \
  --data-urlencode 'query=up'
```

**Range Query**:
```bash
curl -G 'http://localhost:9090/api/v1/query_range' \
  --data-urlencode 'query=up' \
  --data-urlencode 'start=1705329600' \
  --data-urlencode 'end=1705329900' \
  --data-urlencode 'step=15s'
```

**Query with timeout**:
```bash
curl -G 'http://localhost:9090/api/v1/query' \
  --data-urlencode 'query=rate(requests[5m])' \
  --data-urlencode 'timeout=30s'
```

## Common Mistakes & Fixes

| Mistake | Fix |
|---------|-----|
| `up[5]` | Use duration: `up[5m]` |
| `up {job="x"}` | No space: `up{job="x"}` |
| `rate(gauge[5m])` | Use with counters: `rate(counter[5m])` |
| `query too long` | Reduce time range or add filters |
| `not a number (NaN)` | Check metric exists: `metric != 0` |

## Performance Tips

| Tip | Reason |
|-----|--------|
| Add label filters | Reduces data scanned |
| Use short ranges | `[5m]` faster than `[30d]` |
| Aggregate early | `sum()` before division |
| Avoid complex regex | `=~` is slower than `=` |
| Use irate for spikes | `irate()` detects sudden changes |

## Quick Reference Template

```promql
# Check health
up{job="node"}

# Calculate rate
rate(metric_total[5m])

# Convert units
metric_bytes / 1024 / 1024

# Percentage
(part / total) * 100

# Filter & aggregate
sum(rate(metric{instance="host1"}[5m]))

# Time range
metric[5m]
metric[1h]
metric[7d]
```

---

**Keep this open while writing PromQL queries!**
