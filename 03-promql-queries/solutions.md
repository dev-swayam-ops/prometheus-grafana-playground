# Solutions: PromQL Queries

## Solution 1: Simple Instant Vector
**Query**:
```promql
up
```

**Explanation**: 
- Returns current value of `up` metric for all targets
- `up=1` means target is reachable (UP)
- `up=0` means target is unreachable (DOWN)

**Expected Output**:
```
up{instance="prometheus:9090", job="prometheus"} 1
up{instance="node-exporter:9100", job="node"} 1
```

---

## Solution 2: Filter by Label
**Query**:
```promql
up{job="node"}
```

**Curl Command**:
```bash
curl -G 'http://localhost:9090/api/v1/query' \
  --data-urlencode 'query=up{job="node"}'
```

**Explanation**: 
- `{job="node"}` filters metrics by job label
- Only returns Node Exporter's health metric
- Useful for targeting specific services

**Expected Output**:
```
up{instance="node-exporter:9100", job="node"} 1
```

---

## Solution 3: Use Comparison Operator
**Query**:
```promql
node_memory_MemAvailable_bytes > 1000000000
```

**Explanation**: 
- Comparison operators: `>`, `<`, `==`, `!=`, `>=`, `<=`
- Only returns metrics matching condition
- If memory < 1GB, returns empty result

**Expected Output**:
```
node_memory_MemAvailable_bytes{...} 8589934592
```
(Only if > 1GB available)

---

## Solution 4: Query Time Range
**Query**:
```promql
up[5m]
```

**Explanation**: 
- `[5m]` returns values from last 5 minutes
- Shows all data points collected in that range
- Used with functions: `rate()`, `increase()`, `avg_over_time()`

**Expected Output**:
```
up{instance="prometheus:9090", job="prometheus"}
  1 @1705329600
  1 @1705329615
  1 @1705329630
  ...
```

---

## Solution 5: Calculate Rate
**Query**:
```promql
rate(node_network_receive_bytes_total[5m])
```

**Explanation**: 
- `rate()` calculates per-second change over 5 minutes
- Works only with counters (always increasing)
- Result is bytes/second
- Useful for trending/alerting on throughput

**Expected Output**:
```
{device="eth0", instance="...", job="node"} 12345.67
{device="eth1", instance="...", job="node"} 500.23
```

---

## Solution 6: Arithmetic Operations
**Query**:
```promql
node_memory_MemTotal_bytes / 1024 / 1024 / 1024
```

**Explanation**: 
- Converts bytes to GB
- 1 GB = 1024³ bytes
- Operators: `+`, `-`, `*`, `/`, `%`, `^`
- Works on all metric values

**Expected Output**:
```
16.0
```
(Or 8.0, 32.0, etc. depending on system)

---

## Solution 7: Sum Aggregation
**Query**:
```promql
sum(rate(node_network_receive_bytes_total[5m]))
```

**Explanation**: 
- `sum()` aggregates multiple time series into one
- Adds up values across all labels
- Other aggregations: `avg()`, `min()`, `max()`, `count()`
- Useful for total/overall metrics

**Expected Output**:
```
12845.90
```
(Total bytes/sec across all devices)

---

## Solution 8: Count Healthy Targets
**Query**:
```promql
count(up == 1)
```

**Explanation**: 
- `count()` counts number of matching metrics
- `== 1` filters to only UP targets
- Returns single scalar value
- Useful for alerting on target count

**Expected Output**:
```
2
```

---

## Solution 9: Average Function
**Query**:
```promql
avg(node_cpu_seconds_total)
```

**Explanation**: 
- `avg()` calculates average of all matching metrics
- Removes labels in result
- Other options: `avg_over_time()` for value history
- Useful for normalizing across CPUs/instances

**Expected Output**:
```
avg{} 123456.78
```

---

## Solution 10: Complex CPU Idle Percentage
**Query**:
```promql
sum(rate(node_cpu_seconds_total{mode="idle"}[5m])) / sum(rate(node_cpu_seconds_total[5m])) * 100
```

**Breakdown**:
```
Numerator:   sum(rate(...{mode="idle"}...))   = idle CPU seconds/sec
Denominator: sum(rate(...))                    = total CPU seconds/sec
Division:    idle / total                      = idle fraction
Multiply:    * 100                             = percentage
```

**Explanation**: 
- Calculates CPU idle percentage
- `mode="idle"` filters to idle CPU time
- Formula: (idle / total) × 100
- Complex query combining multiple operations

**Expected Output**:
```
85.5
```
(85.5% CPU idle)

---

**All Solutions Complete ✅**
