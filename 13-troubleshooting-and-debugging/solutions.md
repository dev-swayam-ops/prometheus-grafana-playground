# Solutions: Troubleshooting and Debugging

## Solution 1: Check Target Status

**Steps**:
1. Navigate to: `http://localhost:9090/targets`
2. See table with columns: Job, Instance, Status, Last Scrape, Scrape Duration
3. Observe each row

**Output Example**:
```
Job: node-exporter
Instance: localhost:9100
Status: UP (green)
Last Scrape: 2024-01-15 10:30:45
Scrape Duration: 50ms

Job: prometheus
Instance: localhost:9090
Status: UP (green)
Last Scrape: 2024-01-15 10:30:47
Scrape Duration: 10ms
```

**If DOWN**:
- Red indicator
- Check exporter running
- Check firewall/network
- Check Prometheus logs

---

## Solution 2: Test Target Connectivity

**Command**:
```bash
curl -I http://localhost:9100/metrics
```

**Expected Output**:
```
HTTP/1.1 200 OK
Content-Type: text/plain; version=0.0.4
Content-Length: 12345
```

**If Connection Refused**:
```bash
# Exporter not running
docker ps | grep exporter

# Start exporter
docker run -d -p 9100:9100 prom/node-exporter
```

**If Timeout**:
```bash
# Check port listening
netstat -tlnp | grep 9100

# Check firewall
firewall-cmd --list-ports
```

**Explanation**: Tests basic connectivity to exporter endpoint.

---

## Solution 3: Query Prometheus Metrics

**Query 1: Check UP status**:
```promql
up
```

**Output**:
```
up{instance="localhost:9090", job="prometheus"} 1
up{instance="localhost:9100", job="node-exporter"} 1
```
(1 = UP, 0 = DOWN)

**Query 2: Request rate**:
```promql
rate(up[5m])
```

**Output**:
```
Shows how much "up" metric changes over 5 min
(Usually 0, metric stable)
```

---

## Solution 4: View Prometheus Logs

**For Docker**:
```bash
docker logs prometheus | tail -50
```

**For Systemd**:
```bash
journalctl -u prometheus -n 50 -f
```

**Common Errors**:
```
- "failed to parse scrape config": YAML syntax error
- "cannot connect to target": Network/firewall issue
- "invalid scrape interval": Too low or high value
- "out of memory": Cardinality too high
```

**Find Specific Issues**:
```bash
# Search for errors
docker logs prometheus | grep ERROR

# Search for specific metric
docker logs prometheus | grep "http_requests"
```

---

## Solution 5: Check Scrape Config

**Command**:
```bash
curl http://localhost:9090/config | jq '.data'
```

**Output**:
```yaml
global:
  scrape_interval: 15s

scrape_configs:
- job_name: node-exporter
  static_configs:
  - targets:
    - localhost:9100
  scrape_interval: 30s
```

**Verify**:
- job_name exists
- targets list non-empty
- scrape_interval reasonable (15s-60s)
- relabel_configs valid syntax

---

## Solution 6: Test Exporter Metrics Output

**Command**:
```bash
curl http://localhost:9100/metrics | head -100
```

**Output Example**:
```
# HELP node_cpu_seconds_total Seconds the cpus spent in each mode
# TYPE node_cpu_seconds_total counter
node_cpu_seconds_total{cpu="0",mode="idle"} 1000.5
node_cpu_seconds_total{cpu="0",mode="system"} 50.2

# HELP node_memory_MemFree_bytes Memory information
# TYPE node_memory_MemFree_bytes gauge
node_memory_MemFree_bytes 1000000
```

**Check**:
- HELP comments exist
- TYPE line present
- Metric values numeric
- Consistent format

---

## Solution 7: Reload Prometheus Config

**Steps**:
```bash
# 1. Edit prometheus.yml
vim prometheus.yml
# Add new target

# 2. Validate syntax
prometheus --config.file=prometheus.yml --dry-run

# 3. Reload (no downtime)
curl -X POST http://localhost:9090/-/reload

# 4. Verify
curl http://localhost:9090/targets
# New target should appear in 30-60s
```

**If Reload Fails**:
```bash
# Check syntax
prometheus --config.file=prometheus.yml --dry-run

# View error
curl http://localhost:9090/config  # may show old config
docker logs prometheus | grep error
```

---

## Solution 8: Investigate Missing Metric

**Diagnosis Process**:
```bash
# Step 1: Check exporter has metric
curl http://localhost:9100/metrics | grep http_requests_total

# If not found: Instrumentation not in place
# If found: Continue to step 2

# Step 2: Query Prometheus
curl 'http://localhost:9090/api/v1/query?query=http_requests_total'

# If no results: Check scrape config
curl http://localhost:9090/config | grep scrape

# Step 3: Check logs
docker logs prometheus | grep "http_requests"
```

**Common Causes**:
| Cause | Fix |
|-------|-----|
| Exporter not instrumented | Add instrumentation to app |
| Metric path wrong | Verify metrics_path in config |
| Scrape interval too low | Increase to 15s+ |
| Target DOWN | Check connectivity |

---

## Solution 9: Analyze Query Performance

**Browser DevTools**:
1. Open Grafana dashboard
2. Press F12 → Network tab
3. Reload page
4. Look for requests to `/api/datasource/proxy/`
5. Times show query latency

**Optimize Slow Queries**:
```promql
# Bad: Slow query
rate(http_requests_total[5m])  # High cardinality

# Good: Use recording rule
http:request_rate:5m  # Pre-computed
```

**Check Prometheus Slow Log**:
```bash
docker logs prometheus | grep "slow query"
```

---

## Solution 10: Check Storage and Retention

**Command**:
```bash
# Disk usage
du -sh /prometheus/

# Retention setting
curl http://localhost:9090/api/v1/status/config | jq '.data.storage'

# Query oldest data
curl 'http://localhost:9090/api/v1/query?query=up' | jq '.data.result[0].value[0]'
```

**Calculate Storage Needs**:
```
Metrics per second: 10,000
Retention: 30 days = 2,592,000 seconds
Size per metric: ~2KB (estimate)
Total: 10,000 × 2,592,000 × 2KB = ~52GB
```

**Adjust Retention**:
```yaml
prometheus:
  args:
    - --storage.tsdb.retention.time=30d
    - --storage.tsdb.retention.size=100GB
```

---

**All Solutions Complete ✅**
