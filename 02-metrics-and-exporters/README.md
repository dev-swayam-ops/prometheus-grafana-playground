# 02 - Metrics and Exporters

## What You'll Learn
- What are exporters and why they're needed
- Running Node Exporter (system metrics)
- Configuring Prometheus to scrape exporters
- Understanding exporter metrics
- Using Docker Compose for multi-container setup
- Best practices for metric collection

## Prerequisites
- [01-prometheus-basics](../01-prometheus-basics/README.md) completed
- Docker and Docker Compose installed
- Understanding of Prometheus architecture

## Key Concepts

### What is an Exporter?
An exporter is a standalone service that:
1. Collects metrics from a system/application
2. Exposes metrics in Prometheus format
3. Prometheus scrapes it regularly

```
System → Exporter → /metrics endpoint → Prometheus scrapes
```

### Popular Exporters

| Exporter | Metrics | Port | Purpose |
|----------|---------|------|---------|
| **Node Exporter** | CPU, RAM, Disk, Network | 9100 | Linux/Unix systems |
| **MySQL Exporter** | Queries, connections, replication | 9104 | MySQL databases |
| **PostgreSQL Exporter** | Transactions, indexes, locks | 9187 | PostgreSQL databases |
| **Redis Exporter** | Memory, connections, commands | 9121 | Redis cache |
| **Apache Exporter** | Connections, requests, uptime | 9117 | Apache web server |
| **NGINX Exporter** | Requests, connections, status | 9113 | NGINX web server |

### Exporter Metrics Format

Prometheus expects metrics in this format:
```
# HELP metric_name Description
# TYPE metric_name counter|gauge|histogram|summary
metric_name{label1="value1", label2="value2"} 123.45
```

**Example**:
```
# HELP node_cpu_seconds_total Seconds the CPU spent in each mode
# TYPE node_cpu_seconds_total counter
node_cpu_seconds_total{cpu="0",mode="idle"} 120000.5
node_cpu_seconds_total{cpu="0",mode="system"} 5000.2
```

## Hands-on Lab: Node Exporter Setup

### Step 1: Create docker-compose.yml
```yaml
version: '3'
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - --config.file=/etc/prometheus/prometheus.yml
    depends_on:
      - node-exporter

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
    command:
      - --path.rootfs=/host
      - --path.procfs=/host/proc
      - --path.sysfs=/host/sys
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
```

### Step 2: Create prometheus.yml
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']
```

### Step 3: Start Services
```bash
docker-compose up -d
```

**Expected Output**:
```
Creating prometheus ... done
Creating node-exporter ... done
```

### Step 4: Verify Node Exporter is Running
```bash
curl http://localhost:9100/metrics | head -20
```

**Expected Output**:
```
# HELP node_cpu_seconds_total Seconds the CPU spent in each mode
# TYPE node_cpu_seconds_total counter
node_cpu_seconds_total{cpu="0",mode="idle"} 123456.78
...
```

### Step 5: Check Prometheus Targets
Visit: `http://localhost:9090/targets`

**Expected**: Both prometheus and node jobs show State=UP (green)

### Step 6: Query Node Metrics
In Prometheus UI → Graph tab, enter:
```promql
node_memory_MemAvailable_bytes
```

**Expected Output**:
```
node_memory_MemAvailable_bytes{instance="node-exporter:9100", job="node"} 8589934592
```

### Step 7: Explore More Metrics
Try these queries:
```promql
node_cpu_seconds_total
node_disk_io_time_seconds_total
node_network_receive_bytes_total
```

### Step 8: View Raw Metrics
```bash
curl http://localhost:9100/metrics | grep node_cpu_seconds_total | head -5
```

**Expected Output**:
```
node_cpu_seconds_total{cpu="0",mode="idle"} 456789.12
node_cpu_seconds_total{cpu="0",mode="system"} 12345.67
```

## Validation

Verify setup:
```bash
# Check containers running
docker-compose ps

# Check Prometheus targets
curl http://localhost:9090/api/v1/targets | jq '.data.activeTargets[] | {job, state}'

# Query a metric
curl -G 'http://localhost:9090/api/v1/query' \
  --data-urlencode 'query=node_memory_MemAvailable_bytes'
```

**Expected**: All containers running, all targets UP, metric values returned.

## Cleanup

```bash
docker-compose down
docker-compose rm -v
```

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| Node exporter shows DOWN | Ensure `node-exporter:9100` is correct hostname/port |
| No node metrics visible | Wait 15+ seconds for first scrape |
| Permission denied reading /proc | Run with `--privileged` flag |
| Exporter not exposing metrics | Check `/metrics` endpoint is accessible |
| Wrong metric names | Use `/metrics` to discover available metrics |

## Troubleshooting

**Issue**: Node Exporter not reachable
```bash
# Check if running
docker ps | grep node-exporter

# Check logs
docker logs node-exporter

# Test directly
curl http://localhost:9100/metrics
```

**Issue**: Prometheus can't scrape exporter
```bash
# Test connectivity from Prometheus container
docker exec prometheus curl http://node-exporter:9100/metrics

# Check config
docker exec prometheus cat /etc/prometheus/prometheus.yml
```

**Issue**: Metrics showing strange values
```bash
# Check metric type
curl http://localhost:9100/metrics | grep "^# TYPE"

# Verify labels
curl http://localhost:9100/metrics | grep node_memory | head -3
```

## Next Steps

✅ Exporters mastered!

**Ready to proceed to**:
- [03-promql-queries](../03-promql-queries/README.md) - Master PromQL querying
- [04-recording-rules-and-alerting](../04-recording-rules-and-alerting/README.md) - Recording rules
- [exercises](./exercises.md) - Practice with hands-on exercises
