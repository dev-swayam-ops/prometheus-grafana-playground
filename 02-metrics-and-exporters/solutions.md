# Solutions: Metrics and Exporters

## Solution 1: Access Node Exporter Metrics
**Command**:
```bash
curl http://localhost:9100/metrics | head -20
```

**Explanation**: 
- Accesses Node Exporter's `/metrics` endpoint
- `| head -20` shows first 20 lines
- Metrics are in Prometheus text format

**Expected Output**:
```
# HELP node_arp_entries ARP entries by device
# TYPE node_arp_entries gauge
node_arp_entries{device="eth0"} 2
node_arp_entries{device="eth1"} 1
# HELP node_boot_time_seconds ...
```

---

## Solution 2: Find CPU Metrics
**Command**:
```bash
curl http://localhost:9100/metrics | grep "node_cpu"
```

**Explanation**: 
- Filters only CPU-related metrics
- `grep` finds lines containing "node_cpu"
- Shows all CPU metrics across all CPU cores/modes

**Expected Output**:
```
node_cpu_seconds_total{cpu="0",mode="idle"} 456789.12
node_cpu_seconds_total{cpu="0",mode="system"} 12345.67
node_cpu_seconds_total{cpu="0",mode="user"} 34567.89
node_cpu_seconds_total{cpu="1",mode="idle"} 445678.90
...
```

---

## Solution 3: Count Available Metrics
**Command**:
```bash
curl http://localhost:9100/metrics | grep "^node_" | wc -l
```

**Explanation**: 
- `^node_` matches lines starting with "node_"
- `wc -l` counts number of lines
- Excludes `# HELP` and `# TYPE` comments

**Expected Output**:
```
150
```
(Number varies by system, typically 100-300 lines)

---

## Solution 4: Parse Memory Metrics
**Command**:
```bash
curl http://localhost:9100/metrics | grep "node_memory"
```

**Explanation**: 
- Filters memory-related metrics
- Shows available, free, total, cached memory in bytes
- All values in bytes (divide by 1024^3 for GB)

**Expected Output**:
```
node_memory_MemTotal_bytes 17179869184
node_memory_MemFree_bytes 8589934592
node_memory_MemAvailable_bytes 12884901888
node_memory_Buffers_bytes 268435456
node_memory_Cached_bytes 2147483648
```

---

## Solution 5: Run Docker Compose Setup
**Steps**:
```bash
# 1. Create prometheus.yml
cat > prometheus.yml << 'EOF'
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']
EOF

# 2. Create docker-compose.yml (from README)
# 3. Start services
docker-compose up -d

# 4. Verify
docker-compose ps
```

**Explanation**: 
- Defines two scrape jobs (prometheus and node)
- Docker Compose uses service names for networking
- `-d` runs in background

**Expected Output**:
```
NAME              IMAGE                      STATUS
prometheus        prom/prometheus:latest     Up 10 seconds
node-exporter     prom/node-exporter:latest  Up 10 seconds
```

---

## Solution 6: Query Node Metrics in Prometheus
**Steps**:
1. Open browser: `http://localhost:9090`
2. Click "Graph" tab
3. Query box: `node_memory_MemTotal_bytes`
4. Click "Execute"

**Explanation**: 
- PromQL query for total system memory
- Returns value in bytes
- Prometheus waits ~15 seconds for first scrape

**Expected Output**:
```
node_memory_MemTotal_bytes{instance="node-exporter:9100", job="node"} 17179869184
```

---

## Solution 7: Filter Metrics by Job
**Query**:
```promql
up{job="node"}
```

**Command (API)**:
```bash
curl -G 'http://localhost:9090/api/v1/query' \
  --data-urlencode 'query=up{job="node"}'
```

**Explanation**: 
- `up` metric indicates target health
- `{job="node"}` filters to Node Exporter only
- Value 1 = UP, 0 = DOWN

**Expected Output**:
```
up{instance="node-exporter:9100", job="node"} 1
```

---

## Solution 8: Calculate Memory Percentage
**Query**:
```promql
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100
```

**Explanation**: 
- Calculates: (Used Memory / Total Memory) * 100
- Returns percentage between 0-100
- Works with arithmetic operators in PromQL

**Expected Output**:
```
45.3
```
(Approximately 45% memory used)

---

## Solution 9: Check Exporter Metrics Format
**Command**:
```bash
curl http://localhost:9100/metrics | grep "^# TYPE" | head -10
```

**Explanation**: 
- `^# TYPE` matches TYPE declaration lines
- Shows metric type: counter, gauge, histogram, summary
- Helps understand metric behavior

**Expected Output**:
```
# TYPE node_arp_entries gauge
# TYPE node_boot_time_seconds gauge
# TYPE node_cpu_seconds_total counter
# TYPE node_disk_io_now gauge
# TYPE node_disk_io_time_seconds_total counter
# TYPE node_disk_io_time_weighted_seconds_total counter
# TYPE node_disk_reads_completed_total counter
# TYPE node_disk_reads_merged_total counter
# TYPE node_disk_read_bytes_total counter
# TYPE node_disk_read_time_seconds_total counter
```

---

## Solution 10: List Available Labels
**Method 1 - Direct View**:
```bash
curl http://localhost:9100/metrics | grep "node_" | grep "{" | head -20
```

**Method 2 - Via Prometheus API**:
```bash
curl 'http://localhost:9090/api/v1/labels'
```

**Explanation**: 
- Method 1: Shows metrics with their label values
- Method 2: Returns list of all label names in Prometheus
- Labels are key-value pairs identifying metric dimensions

**Expected Output** (Method 1):
```
node_cpu_seconds_total{cpu="0",mode="idle"} 456789.12
node_cpu_seconds_total{cpu="0",mode="system"} 12345.67
node_cpu_seconds_total{cpu="1",mode="idle"} 445678.90
```

**Expected Output** (Method 2):
```json
{
  "status": "success",
  "data": [
    "__name__",
    "cpu",
    "device",
    "instance",
    "job",
    "mode",
    ...
  ]
}
```

---

**All Solutions Complete ✅**
