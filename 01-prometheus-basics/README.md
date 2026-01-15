# 01 - Prometheus Basics

## What You'll Learn
- Prometheus architecture and components
- Core concepts: metrics, labels, and time-series data
- Scraping targets and pull-based monitoring
- Running Prometheus in Docker
- Understanding the TSDB (Time-Series Database)
- Basic metric collection and exploration

## Prerequisites
- [00-setup-and-prerequisites](../00-setup-and-prerequisites/README.md) completed
- Docker and Docker Compose installed
- Basic understanding of monitoring concepts

## Key Concepts

### Prometheus Architecture
```
Prometheus Server
├── Scraper (pulls metrics from targets)
├── TSDB (stores time-series data)
├── HTTP API (query interface)
└── Alerting Manager (sends alerts)

Data Flow: Target → Scraper → TSDB → Queries
```

### Core Terminology

| Term | Definition | Example |
|------|-----------|---------|
| **Metric** | A measurement with a name and labels | `http_requests_total` |
| **Labels** | Key-value pairs that identify dimensions | `job="api", instance="localhost:8000"` |
| **Time-Series** | Metric + labels + timestamp + value | `http_requests_total{job="api"} 1024 1234567890` |
| **Scraping** | Pull-based collection from endpoints | Prometheus polls `/metrics` endpoint |
| **Target** | An application exposing metrics | Node exporter, app server |
| **Instance** | A target's unique address | `localhost:9090` |
| **Job** | A group of related targets | `prometheus`, `node`, `mysql` |

### Metric Types

**Counter**: Only increases (never decreases)
```
# Example: http_requests_total
http_requests_total{method="GET"} 1000
http_requests_total{method="POST"} 50
```

**Gauge**: Can increase or decrease
```
# Example: node_memory_MemFree_bytes
node_memory_MemFree_bytes 4294967296
```

**Histogram**: Bucket-based distribution
```
# Example: http_request_duration_seconds
http_request_duration_seconds_bucket{le="0.1"} 100
http_request_duration_seconds_bucket{le="0.5"} 450
```

**Summary**: Percentile-based distribution
```
# Example: temperature
temperature{quantile="0.5"} 72
temperature{quantile="0.99"} 85
```

## Hands-on Lab: Run Prometheus in Docker

### Step 1: Create Prometheus Configuration
Create `prometheus.yml`:
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
```

### Step 2: Run Prometheus Container
```bash
docker run -d \
  --name prometheus \
  -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus:latest
```

**Expected Output**:
```
Container ID: abc123def456...
```

### Step 3: Verify Prometheus is Running
```bash
docker ps | grep prometheus
```

**Expected Output**:
```
abc123def456  prom/prometheus  "prometheus --config"  9090/tcp  prometheus
```

### Step 4: Access Prometheus Web UI
Open browser: `http://localhost:9090`

**Expected**:
- Prometheus logo
- Graph tab
- Targets tab (1 target: Prometheus itself)

### Step 5: Query Prometheus
Use Graph tab, enter query:
```promql
up{job="prometheus"}
```

**Expected Output**:
```
up{job="prometheus"} 1 (green indicator, value=1 means running)
```

### Step 6: Explore Metrics
Try these queries:
```promql
# Process memory usage
process_resident_memory_bytes

# Scrape duration
scrape_duration_seconds

# Target scrape interval
scrape_interval
```

### Step 7: Check Targets
Click "Targets" tab in UI.

**Expected**:
- Job: `prometheus`
- Instance: `localhost:9090`
- State: UP (green)
- Scrape Duration: ~10ms

### Step 8: View Config
Click "Status" → "Configuration" in UI.

**Expected**: Your `prometheus.yml` content displayed.

## Validation

Verify your setup:
```bash
# Check container running
docker ps | grep prometheus

# Check logs
docker logs prometheus | tail -20

# Query via API
curl http://localhost:9090/api/v1/query?query=up
```

**Expected**: Container running, logs show "Ready to serve", API returns JSON with metrics.

## Cleanup

To stop Prometheus:
```bash
docker stop prometheus
docker rm prometheus
```

To remove volume (keeps data):
```bash
docker volume rm prometheus_data
```

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| Port 9090 already in use | Change: `docker run -p 9091:9090` |
| Config file not found | Use absolute path: `-v /full/path/prometheus.yml` |
| "Error scraping target" | Ensure target is running and accessible |
| No metrics appearing | Wait 15 seconds (scrape interval) |
| High memory usage | Reduce `scrape_interval` or storage retention |

## Troubleshooting

**Issue**: Prometheus shows "ServiceUnavailable" for targets
```bash
# Check if target is reachable
curl http://localhost:9100/metrics

# Verify config YAML syntax
docker run --rm -v $(pwd)/prometheus.yml:/prometheus.yml \
  prom/prometheus --config.file=/prometheus.yml --dry-run
```

**Issue**: Metrics not updating
```bash
# Check scrape interval
# View in UI: Status → Configuration

# Restart Prometheus
docker restart prometheus

# Check target health
curl http://localhost:9090/api/v1/targets
```

**Issue**: Too much disk usage
```bash
# Reduce retention period
docker run -d --name prometheus \
  prom/prometheus:latest \
  --storage.tsdb.retention.time=7d
```

## Next Steps

✅ Prometheus basics mastered!

**Ready to proceed to**:
- [02-metrics-and-exporters](../02-metrics-and-exporters/README.md) - Collect metrics from applications
- [03-promql-queries](../03-promql-queries/README.md) - Advanced querying techniques
- [exercises](./exercises.md) - Practice with hands-on exercises
