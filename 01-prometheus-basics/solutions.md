# Solutions: Prometheus Basics

## Solution 1: Start Prometheus Container
**Commands**:
```bash
docker run -d --name prometheus -p 9090:9090 prom/prometheus:latest
docker ps | grep prometheus
```

**Explanation**: 
- `-d` runs container in background
- `--name prometheus` assigns a name
- `-p 9090:9090` maps port for web access
- `prom/prometheus` is the official image

**Expected Output**:
```
Container ID hash...
abc123def456  prom/prometheus  "prometheus --config"  9090/tcp  prometheus
```

---

## Solution 2: Access Prometheus Web UI
**Method 1 - Browser**:
```
http://localhost:9090
```

**Method 2 - Command Line**:
```bash
curl http://localhost:9090
```

**Explanation**: 
Prometheus exposes HTTP API on port 9090. The UI is served at root path.

**Expected Output**:
```
HTTP/1.1 200 OK
Content-Type: text/html
(HTML with Prometheus UI)
```

---

## Solution 3: Query the `up` Metric
**Steps**:
1. Visit: `http://localhost:9090`
2. Click: "Graph" tab
3. Query box: `up`
4. Click: "Execute" button

**Explanation**: 
- `up` is a built-in metric showing if a target is reachable (1=UP, 0=DOWN)
- Prometheus automatically exposes its own metrics

**Expected Output**:
```
up{job="prometheus", instance="localhost:9090"} 1
```

---

## Solution 4: Inspect Prometheus Metrics
**Command via API**:
```bash
curl 'http://localhost:9090/api/v1/query?query=up'
```

**Command via Web**:
```bash
curl 'http://localhost:9090/api/v1/label/__name__/values'
```

**Explanation**: 
- `/api/v1/query` executes a PromQL query
- `/api/v1/label/__name__/values` lists all metric names

**Expected Output**:
```json
{
  "status": "success",
  "data": [
    "up",
    "process_resident_memory_bytes",
    "go_goroutines",
    ...
  ]
}
```

---

## Solution 5: Configure Custom Prometheus
**Create prometheus.yml**:
```yaml
global:
  scrape_interval: 10s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

**Run Prometheus with Config**:
```bash
docker stop prometheus && docker rm prometheus

docker run -d --name prometheus \
  -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus:latest
```

**Explanation**: 
- `prometheus.yml` defines scrape targets and intervals
- `-v` flag mounts config file into container
- `/etc/prometheus/prometheus.yml` is default path inside container

**Validation**: 
- Open UI → Status → Configuration
- Should show your custom config

---

## Solution 6: Query Process Metrics
**Command (UI)**:
```
Query box: process_resident_memory_bytes
Execute
```

**Command (CLI)**:
```bash
curl -G 'http://localhost:9090/api/v1/query' \
  --data-urlencode 'query=process_resident_memory_bytes'
```

**Explanation**: 
- This metric shows Prometheus's own memory usage
- Units are bytes
- Prometheus exposes Go runtime metrics by default

**Expected Output**:
```
process_resident_memory_bytes{job="prometheus"} 50331648
```

---

## Solution 7: Use Label Filtering
**Command (UI)**:
```
Query box: up{job="prometheus"}
Execute
```

**Command (CLI)**:
```bash
curl -G 'http://localhost:9090/api/v1/query' \
  --data-urlencode 'query=up{job="prometheus"}'
```

**Explanation**: 
- `{job="prometheus"}` filters metrics by label
- Only returns metrics where job label = "prometheus"
- Syntax: `metric_name{label1="value1", label2="value2"}`

**Expected Output**:
```
up{job="prometheus", instance="localhost:9090"} 1
```

---

## Solution 8: View Target Status
**Steps**:
1. Open: `http://localhost:9090`
2. Click: "Targets" tab
3. Observe:
   - **Job**: prometheus
   - **Instance**: localhost:9090
   - **State**: UP (green indicator)
   - **Last Scrape**: Recent timestamp
   - **Scrape Duration**: ~5-15ms

**Explanation**: 
- Targets tab shows all configured scrape targets
- Green = UP (successfully scraped)
- Red = DOWN (unreachable)
- Shows scrape success rate and timing

---

## Solution 9: Query via HTTP API
**Command**:
```bash
curl -G 'http://localhost:9090/api/v1/query' \
  --data-urlencode 'query=process_resident_memory_bytes'
```

**Alternative - Store Results**:
```bash
curl 'http://localhost:9090/api/v1/query?query=up' > metrics.json
cat metrics.json | jq .
```

**Explanation**: 
- Prometheus HTTP API enables programmatic access
- `-G` flag for GET request
- `--data-urlencode` safely encodes query parameter
- Response is JSON with status, result type, and values

**Expected Output**:
```json
{
  "status": "success",
  "data": {
    "resultType": "vector",
    "result": [
      {
        "metric": {
          "job": "prometheus",
          "instance": "localhost:9090"
        },
        "value": [1705329600, "1"]
      }
    ]
  }
}
```

---

## Solution 10: Check Configuration Syntax
**Method 1 - Dry Run**:
```bash
docker run --rm -v $(pwd)/prometheus.yml:/prometheus.yml \
  prom/prometheus --config.file=/prometheus.yml --dry-run
```

**Method 2 - Check Logs**:
```bash
docker logs prometheus 2>&1 | grep -iE "config|error|listening" | head -10
```

**Explanation**: 
- `--dry-run` validates config without running server
- Logs show startup errors if config is invalid
- YAML syntax must be exact (spacing, indentation matter)

**Expected Output** (Success):
```
config file loaded successfully
Server is ready to receive web requests
listening on: 0.0.0.0:9090
```

**Expected Output** (Error):
```
error loading config file: yaml: line X: mapping values are not allowed in this context
```

---

**All Solutions Complete ✅**
