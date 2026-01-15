# Exercises: Prometheus Basics

## Exercise 1: Start Prometheus Container (Easy)
Run Prometheus in Docker with default configuration.

**Task**:
```bash
docker run -d --name prometheus -p 9090:9090 prom/prometheus:latest
docker ps | grep prometheus
```

**Validation**: Container running, listening on port 9090.

---

## Exercise 2: Access Prometheus Web UI (Easy)
Open Prometheus UI and verify it's accessible.

**Task**:
```bash
# Open in browser
http://localhost:9090

# Or curl
curl http://localhost:9090
```

**Validation**: HTTP 200 response, Prometheus logo visible in browser.

---

## Exercise 3: Query the `up` Metric (Easy)
Query the built-in `up` metric showing target health.

**Task**:
1. Open Prometheus UI → Graph tab
2. Enter query: `up`
3. Click "Execute"

**Expected Output**:
```
up{job="prometheus"} 1
```

**Validation**: Shows value of 1 (target is UP).

---

## Exercise 4: Inspect Prometheus Metrics (Easy)
View available metrics in Prometheus.

**Task**:
```bash
# Via API
curl 'http://localhost:9090/api/v1/query?query=up'

# Or visit in browser
http://localhost:9090/api/v1/label/__name__/values
```

**Validation**: Returns JSON with metric names and values.

---

## Exercise 5: Configure Custom Prometheus (Medium)
Create custom `prometheus.yml` and run Prometheus with it.

**Task**:
Create `prometheus.yml`:
```yaml
global:
  scrape_interval: 10s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

Run:
```bash
docker stop prometheus && docker rm prometheus
docker run -d --name prometheus \
  -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus:latest
```

**Validation**: Prometheus starts, shows in UI Status → Configuration.

---

## Exercise 6: Query Process Metrics (Medium)
Query Prometheus's own process metrics.

**Task**:
1. Open Prometheus UI → Graph tab
2. Enter: `process_resident_memory_bytes`
3. Click "Execute"

**Expected Output**:
```
process_resident_memory_bytes{job="prometheus"} 123456789
```

**Validation**: Returns a number > 0 (memory usage in bytes).

---

## Exercise 7: Use Label Filtering (Medium)
Query metrics with label filtering.

**Task**:
1. Open Prometheus UI → Graph tab
2. Enter: `up{job="prometheus"}`
3. Click "Execute"

**Expected Output**:
```
up{job="prometheus"} 1
```

**Validation**: Only returns prometheus job metric.

---

## Exercise 8: View Target Status (Medium)
Check target status and scrape details in UI.

**Task**:
1. Open Prometheus UI
2. Click "Targets" tab
3. Observe:
   - Job name
   - Instance address
   - State (UP/DOWN)
   - Last Scrape
   - Scrape Duration

**Validation**: Shows prometheus target with State=UP.

---

## Exercise 9: Query via HTTP API (Medium)
Query Prometheus using its HTTP API directly.

**Task**:
```bash
curl -G 'http://localhost:9090/api/v1/query' \
  --data-urlencode 'query=process_resident_memory_bytes'
```

**Expected Output**:
```json
{
  "status": "success",
  "data": {
    "resultType": "vector",
    "result": [...]
  }
}
```

**Validation**: Returns valid JSON with metric data.

---

## Exercise 10: Check Prometheus Configuration Syntax (Medium)
Validate your prometheus.yml has correct YAML syntax.

**Task**:
```bash
# Method 1: Dry-run with official image
docker run --rm -v $(pwd)/prometheus.yml:/prometheus.yml \
  prom/prometheus --config.file=/prometheus.yml --dry-run

# Method 2: Check logs
docker logs prometheus | grep -i "config\|error" | head -10
```

**Validation**: No errors, "success" message or configuration parsed correctly.

---

**All Exercises Complete ✅**
