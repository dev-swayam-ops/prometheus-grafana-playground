# Exercises: PromQL Queries

## Exercise 1: Simple Instant Vector (Easy)
Query the `up` metric to check target health.

**Task**:
In Prometheus UI (Graph tab):
```promql
up
```

**Validation**: Shows all targets with their status (1=UP, 0=DOWN).

---

## Exercise 2: Filter by Label (Easy)
Query only Node Exporter's `up` metric.

**Task**:
```promql
up{job="node"}
```

**Validation**: Returns only Node job metric with value 1.

---

## Exercise 3: Use Comparison Operator (Easy)
Find metrics greater than a threshold.

**Task**:
```promql
node_memory_MemAvailable_bytes > 1000000000
```

**Validation**: Returns memory metric only if > 1GB available.

---

## Exercise 4: Query Time Range (Easy)
Get metric values over last 5 minutes.

**Task**:
```promql
up[5m]
```

**Validation**: Shows timestamp and values for each scrape in last 5 minutes.

---

## Exercise 5: Calculate Rate (Medium)
Calculate bytes per second received on network.

**Task**:
```promql
rate(node_network_receive_bytes_total[5m])
```

**Validation**: Returns rate in bytes/second per network device.

---

## Exercise 6: Arithmetic Operations (Medium)
Convert memory bytes to gigabytes.

**Task**:
```promql
node_memory_MemTotal_bytes / 1024 / 1024 / 1024
```

**Validation**: Returns memory in GB (e.g., `16.0`).

---

## Exercise 7: Sum Aggregation (Medium)
Sum total bytes received across all network interfaces.

**Task**:
```promql
sum(rate(node_network_receive_bytes_total[5m]))
```

**Validation**: Returns single number = total bytes/sec.

---

## Exercise 8: Count Healthy Targets (Medium)
Count how many targets are UP.

**Task**:
```promql
count(up == 1)
```

**Validation**: Returns count of healthy targets (e.g., `2`).

---

## Exercise 9: Average Function (Medium)
Calculate average CPU seconds across all CPUs.

**Task**:
```promql
avg(node_cpu_seconds_total)
```

**Validation**: Returns single averaged value.

---

## Exercise 10: Complex Query (Medium)
Calculate CPU idle percentage.

**Task**:
```promql
sum(rate(node_cpu_seconds_total{mode="idle"}[5m])) / sum(rate(node_cpu_seconds_total[5m])) * 100
```

**Validation**: Returns percentage between 0-100 (CPU idle %).

---

**All Exercises Complete ✅**
