# 03 - PromQL Queries

## What You'll Learn
- PromQL syntax and basic queries
- Instant vectors vs. range vectors
- Aggregation and arithmetic operations
- Functions: rate, increase, sum, avg, etc.
- Time range selection
- Query performance and optimization
- Real-world query examples

## Prerequisites
- [01-prometheus-basics](../01-prometheus-basics/README.md) completed
- [02-metrics-and-exporters](../02-metrics-and-exporters/README.md) preferred
- Access to Prometheus instance with metrics

## Key Concepts

### Query Types

**Instant Vector Query**:
- Returns current value at a single point in time
- Example: `up` → `[1, 1, 0]` (latest values only)

**Range Vector Query**:
- Returns values over a time period
- Example: `up[5m]` → Last 5 minutes of data
- Used with aggregation/functions

**Scalar**:
- Single numeric value
- Example: `1024` or `3.14`

### Time Durations
```
s = seconds (e.g., 60s = 1 minute)
m = minutes (e.g., 5m = 5 minutes)
h = hours (e.g., 1h = 1 hour)
d = days (e.g., 7d = 7 days)
w = weeks (e.g., 4w = 4 weeks)
y = years (e.g., 1y = 1 year)
```

### Operators

| Operator | Purpose | Example |
|----------|---------|---------|
| `+` | Addition | `a + b` |
| `-` | Subtraction | `a - b` |
| `*` | Multiplication | `a * b` |
| `/` | Division | `a / b` |
| `%` | Modulo | `a % b` |
| `^` | Power | `a ^ 2` |
| `==` | Equal | `up == 1` |
| `!=` | Not equal | `up != 0` |
| `>` | Greater than | `errors > 100` |
| `<` | Less than | `errors < 50` |
| `>=` | Greater or equal | `cpu > 80` |
| `<=` | Less or equal | `mem <= 90` |

### Label Matching

| Operator | Behavior | Example |
|----------|----------|---------|
| `=` | Exact match | `job="prometheus"` |
| `!=` | Not equal | `job!="backup"` |
| `=~` | Regex match | `job=~"prom.*"` |
| `!~` | Not regex match | `job!~"backup.*"` |

## Hands-on Lab: PromQL Queries

### Prerequisites: Running Prometheus with Metrics
Ensure you have Prometheus running with Node Exporter from [02-metrics-and-exporters](../02-metrics-and-exporters/README.md).

### Step 1: Simple Metric Query
**Query**:
```promql
up
```

**In UI**:
1. Open: `http://localhost:9090`
2. Graph tab → Enter query → Execute

**Expected Output**:
```
up{instance="prometheus:9090", job="prometheus"} 1
up{instance="node-exporter:9100", job="node"} 1
```

### Step 2: Query with Label Filter
**Query**:
```promql
up{job="node"}
```

**Expected Output**:
```
up{instance="node-exporter:9100", job="node"} 1
```

### Step 3: Query with Comparison
**Query**:
```promql
node_memory_MemAvailable_bytes > 1000000000
```

**Expected Output**:
```
node_memory_MemAvailable_bytes{...} 8589934592
```

(Returns only if memory > 1GB)

### Step 4: Calculate Rate of Change
**Query**:
```promql
rate(node_network_receive_bytes_total[5m])
```

**Expected Output**:
```
{device="eth0", instance="...", job="node"} 12345.67
```

(Bytes per second received)

### Step 5: Aggregate Across Instances
**Query**:
```promql
sum(rate(node_network_receive_bytes_total[5m]))
```

**Expected Output**:
```
12345.67
```

(Total bytes/second across all network interfaces)

### Step 6: Use avg() Function
**Query**:
```promql
avg(node_cpu_seconds_total)
```

**Expected Output**:
```
avg{} 123456.78
```

(Average CPU seconds across all CPUs)

### Step 7: Check Target Health
**Query**:
```promql
count(up == 1)
```

**Expected Output**:
```
2
```

(2 targets are healthy)

### Step 8: Arithmetic Operations
**Query**:
```promql
node_memory_MemAvailable_bytes / 1024 / 1024 / 1024
```

**Expected Output**:
```
8.0
```

(Memory in GB instead of bytes)

## Validation

Test queries:
```bash
# Via API
curl -G 'http://localhost:9090/api/v1/query' \
  --data-urlencode 'query=rate(node_network_receive_bytes_total[5m])'

# Check query execution time
curl -G 'http://localhost:9090/api/v1/query' \
  --data-urlencode 'query=up' | jq '.data'
```

## Cleanup

No cleanup needed; queries don't modify data.

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| "No data to graph" | Metric might not exist; check `/metrics` endpoint |
| Wrong range syntax `[5]` | Use duration: `[5m]`, `[1h]`, not just number |
| Rate only for counters | Use `rate(counter[5m])`, not for gauges |
| Forgetting to filter labels | Use `{job="xyz"}` to limit results |
| Query too slow | Reduce time range or use better label matching |

## Troubleshooting

**Issue**: Query returns "No data" or empty
```promql
# Check if metric exists
count(node_cpu_seconds_total) > 0

# Try without filters
node_cpu_seconds_total

# Check time range
up[5m]  # Returns values from last 5 minutes
```

**Issue**: Regex not matching labels
```promql
# Check available label values
{__name__="up"}

# Use correct regex syntax
job=~"prom.*"   # Correct
job=~prom.*     # Wrong (missing quotes)
```

**Issue**: Rate giving negative values
```promql
# Use increase() for adjusted counter
increase(errors_total[5m])

# Or use rate with aggregation
sum(rate(errors_total[5m]))
```

## Next Steps

✅ PromQL mastered!

**Ready to proceed to**:
- [04-recording-rules-and-alerting](../04-recording-rules-and-alerting/README.md) - Recording rules and alerts
- [06-grafana-basics](../06-grafana-basics/README.md) - Visualize queries in Grafana
- [exercises](./exercises.md) - Practice with hands-on exercises
