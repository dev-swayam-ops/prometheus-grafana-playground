# Cheatsheet: Prometheus Basics

## Prometheus Docker Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `docker run -d -p 9090:9090 prom/prometheus` | Start Prometheus | `docker run -d --name prometheus -p 9090:9090 prom/prometheus:latest` |
| `docker ps \| grep prometheus` | List Prometheus container | Shows running Prometheus info |
| `docker logs prometheus` | View logs | `docker logs prometheus -f` |
| `docker stop prometheus` | Stop Prometheus | `docker stop prometheus` |
| `docker rm prometheus` | Remove container | `docker rm prometheus` |
| `docker exec prometheus <cmd>` | Run command in container | `docker exec prometheus cat /etc/prometheus/prometheus.yml` |

## Prometheus Configuration (prometheus.yml)

| Setting | Purpose | Example |
|---------|---------|---------|
| `global.scrape_interval` | How often to scrape targets | `15s` (default) |
| `global.evaluation_interval` | How often to evaluate rules | `15s` (default) |
| `scrape_configs` | Define scrape jobs/targets | See below |
| `job_name` | Name of scrape job | `prometheus`, `node`, `mysql` |
| `static_configs` | Define targets manually | See below |
| `targets` | List of addresses to scrape | `localhost:9090`, `192.168.1.1:8080` |

## Basic prometheus.yml Template

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

## Prometheus API Endpoints

| Endpoint | Purpose | Example |
|----------|---------|---------|
| `/` | Web UI | `http://localhost:9090` |
| `/api/v1/query` | Query metrics | `?query=up` |
| `/api/v1/query_range` | Range queries | `?query=up&start=...&end=...` |
| `/api/v1/labels` | List all labels | `?match[]=up` |
| `/api/v1/label/<name>/values` | Label values | `/api/v1/label/job/values` |
| `/api/v1/targets` | List all targets | Returns scrape targets |
| `/api/v1/rules` | List alert rules | Returns configured rules |
| `/metrics` | Prometheus own metrics | Shows process, go, promhttp metrics |

## Basic PromQL Queries

| Query | Purpose | Example |
|-------|---------|---------|
| `<metric>` | Select metric | `up` |
| `{label="value"}` | Filter by label | `up{job="prometheus"}` |
| `{label=~"regex"}` | Regex label match | `up{job=~"prom.*"}` |
| `{label!="value"}` | Not equal filter | `up{job!="backup"}` |
| `metric > value` | Comparison | `process_resident_memory_bytes > 1000000` |
| `metric [range]` | Range vector | `up[5m]` (last 5 minutes) |
| `rate(counter[range])` | Per-second rate | `rate(http_requests_total[1m])` |
| `increase(counter[range])` | Total increase | `increase(errors_total[1h])` |

## UI Navigation

| Location | Purpose | Access |
|----------|---------|--------|
| **Graph** | Query and visualize metrics | Click "Graph" tab |
| **Targets** | View scrape target status | Click "Targets" tab |
| **Alerts** | View alert rules | Click "Alerts" tab |
| **Status** | Configuration and runtime info | Click "Status" menu |
| **Help** | API documentation | Click "Help" menu |

## Metric Types Reference

| Type | Behavior | Example |
|------|----------|---------|
| **Counter** | Always increases | `http_requests_total` |
| **Gauge** | Can go up or down | `memory_free_bytes` |
| **Histogram** | Bucket-based distribution | `request_duration_seconds_bucket` |
| **Summary** | Percentile-based | `response_time_seconds{quantile="0.95"}` |

## Curl Query Examples

```bash
# Simple metric query
curl 'http://localhost:9090/api/v1/query?query=up'

# Query with filtering
curl 'http://localhost:9090/api/v1/query?query=up{job="prometheus"}'

# Range query (last 1 hour)
curl 'http://localhost:9090/api/v1/query_range?query=up&start=2024-01-15T00:00:00Z&end=2024-01-15T01:00:00Z&step=60s'

# List all targets
curl 'http://localhost:9090/api/v1/targets'

# Get all metrics
curl 'http://localhost:9090/api/v1/label/__name__/values'
```

## Common Metric Naming Conventions

| Convention | Example |
|-----------|---------|
| `_total` | Counter | `http_requests_total` |
| `_seconds` | Timing | `request_duration_seconds` |
| `_bytes` | Size | `memory_usage_bytes` |
| `_percent` | Percentage | `cpu_usage_percent` |
| `_count` | Count | `active_connections_count` |

## Quick Setup Checklist

```bash
# 1. Create config file
cat > prometheus.yml << 'EOF'
global:
  scrape_interval: 15s
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
EOF

# 2. Start Prometheus
docker run -d --name prometheus \
  -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus:latest

# 3. Wait and verify
sleep 3
curl http://localhost:9090/api/v1/query?query=up

# 4. Open UI
open http://localhost:9090
```

---

**Keep this handy while exploring Prometheus!**
