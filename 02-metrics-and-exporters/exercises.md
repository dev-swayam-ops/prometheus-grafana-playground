# Exercises: Metrics and Exporters

## Exercise 1: Access Node Exporter Metrics (Easy)
Access the Node Exporter `/metrics` endpoint and verify it's working.

**Task**:
```bash
curl http://localhost:9100/metrics | head -20
```

**Validation**: Returns metrics starting with `# HELP` and `# TYPE` comments.

---

## Exercise 2: Find CPU Metrics (Easy)
Search for CPU-related metrics in Node Exporter.

**Task**:
```bash
curl http://localhost:9100/metrics | grep "node_cpu"
```

**Validation**: Returns `node_cpu_seconds_total` metric lines.

---

## Exercise 3: Count Available Metrics (Easy)
Count total number of unique metrics exposed.

**Task**:
```bash
curl http://localhost:9100/metrics | grep "^node_" | wc -l
```

**Validation**: Returns a number > 50 (Node Exporter exposes many metrics).

---

## Exercise 4: Parse Memory Metrics (Easy)
Extract memory-related metrics from Node Exporter.

**Task**:
```bash
curl http://localhost:9100/metrics | grep "node_memory"
```

**Validation**: Shows memory metrics like `MemTotal`, `MemFree`, `MemAvailable`.

---

## Exercise 5: Run Docker Compose Setup (Medium)
Create and run a complete Prometheus + Node Exporter setup.

**Task**:
1. Create `docker-compose.yml` from README Step 1
2. Create `prometheus.yml` from README Step 2
3. Run: `docker-compose up -d`
4. Verify: `docker-compose ps`

**Validation**: Both containers running (prometheus, node-exporter).

---

## Exercise 6: Query Node Metrics in Prometheus (Medium)
Use Prometheus UI to query Node Exporter metrics.

**Task**:
1. Open: `http://localhost:9090`
2. Graph tab
3. Enter: `node_memory_MemTotal_bytes`
4. Click Execute

**Validation**: Returns memory total value (typically > 1GB).

---

## Exercise 7: Filter Metrics by Job (Medium)
Query metrics filtered by the Node job.

**Task**:
In Prometheus UI:
```promql
up{job="node"}
```

**Validation**: Returns only Node Exporter's `up` metric with value 1.

---

## Exercise 8: Calculate Memory Percentage (Medium)
Use arithmetic to calculate memory usage percentage.

**Task**:
In Prometheus UI:
```promql
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100
```

**Validation**: Returns a percentage between 0-100.

---

## Exercise 9: Check Exporter Metrics Format (Medium)
Verify the Prometheus metric format is correct.

**Task**:
```bash
curl http://localhost:9100/metrics | grep "^# TYPE" | head -10
```

**Validation**: Shows TYPE declarations like `counter`, `gauge`, `histogram`.

---

## Exercise 10: List Available Labels (Medium)
Find all unique labels used by Node Exporter metrics.

**Task**:
```bash
# Get all metrics with labels
curl http://localhost:9100/metrics | grep "node_" | grep "{" | head -20

# Or via Prometheus API
curl 'http://localhost:9090/api/v1/labels' | jq '.data'
```

**Validation**: Shows labels like `cpu`, `device`, `instance`, `job`, `mode`.

---

**All Exercises Complete ✅**
