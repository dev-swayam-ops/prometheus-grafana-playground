# Cheatsheet: Metrics and Exporters

## Popular Exporters Quick Reference

| Exporter | Docker Image | Port | Typical Metrics |
|----------|--------------|------|-----------------|
| Node Exporter | `prom/node-exporter` | 9100 | CPU, RAM, Disk, Network |
| MySQL Exporter | `prom/mysqld-exporter` | 9104 | Queries, connections, replication |
| PostgreSQL Exporter | `prometheuscommunity/postgres-exporter` | 9187 | Transactions, indexes, lock |
| Redis Exporter | `oliver006/redis_exporter` | 9121 | Memory, keys, commands |
| Apache Exporter | `lusitaniae/apache-exporter` | 9117 | Requests, connections, uptime |
| NGINX Exporter | `nginx/nginx-prometheus-exporter` | 9113 | Requests, connections, upstream |

## Node Exporter Common Metrics

| Metric | Type | Purpose | Example |
|--------|------|---------|---------|
| `node_cpu_seconds_total` | counter | CPU time by mode | `node_cpu_seconds_total{mode="idle"}` |
| `node_memory_MemTotal_bytes` | gauge | Total RAM | Always constant value |
| `node_memory_MemAvailable_bytes` | gauge | Available RAM | Changes dynamically |
| `node_memory_MemFree_bytes` | gauge | Free RAM | Changes dynamically |
| `node_disk_io_time_seconds_total` | counter | Disk I/O time | Used for calculating I/O |
| `node_disk_bytes_read` | counter | Total bytes read | Cumulative counter |
| `node_network_receive_bytes_total` | counter | Network bytes received | Per interface |
| `node_network_transmit_bytes_total` | counter | Network bytes sent | Per interface |
| `node_uname_info` | gauge | System info | Contains kernel version, OS |
| `node_boot_time_seconds` | gauge | Boot timestamp | Unix timestamp |

## Docker Compose Template

```yaml
version: '3'
services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    depends_on:
      - node-exporter

  node-exporter:
    image: prom/node-exporter:latest
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
```

## Scraping Configuration

| Config | Purpose | Example |
|--------|---------|---------|
| `scrape_interval` | How often to scrape | `15s`, `30s`, `1m` |
| `scrape_timeout` | Max time to wait for response | `10s` (default) |
| `static_configs` | Manual target list | See template above |
| `targets` | List of endpoints | `['localhost:9100', 'host2:9100']` |
| `job_name` | Scrape job identifier | `node`, `mysql`, `redis` |

## Exporter Curl Commands

| Task | Command |
|------|---------|
| Check if exporter running | `curl http://localhost:9100/metrics` |
| Count total metrics | `curl http://localhost:9100/metrics \| grep "^node_" \| wc -l` |
| View first 20 lines | `curl http://localhost:9100/metrics \| head -20` |
| Filter specific metric | `curl http://localhost:9100/metrics \| grep node_cpu` |
| Search for label | `curl http://localhost:9100/metrics \| grep "device="` |
| Pretty print (jq) | `curl http://localhost:9100/metrics \| grep node_memory` |

## Prometheus API for Metrics

| Endpoint | Purpose | Example |
|----------|---------|---------|
| `/api/v1/query` | Execute PromQL query | `?query=up` |
| `/api/v1/label/` | List label names | `/api/v1/label/__name__/values` |
| `/api/v1/targets` | List all targets | Shows active/dropped targets |
| `/api/v1/status/config` | View configuration | Returns prometheus.yml content |
| `/metrics` | Prometheus own metrics | `curl http://localhost:9090/metrics` |

## Metric Format (Text Protocol)

```
# HELP <metric_name> <description>
# TYPE <metric_name> <type>
<metric_name>{<label1>="<value1>", <label2>="<value2>"} <number> [<timestamp>]
```

**Example**:
```
# HELP node_cpu_seconds_total Seconds the CPU spent in each mode
# TYPE node_cpu_seconds_total counter
node_cpu_seconds_total{cpu="0",mode="idle"} 120000.5
node_cpu_seconds_total{cpu="0",mode="system"} 5000.2
```

## Docker Commands for Exporters

| Command | Purpose |
|---------|---------|
| `docker run -d -p 9100:9100 prom/node-exporter` | Run Node Exporter standalone |
| `docker logs <container_id>` | View exporter startup logs |
| `docker exec <container> curl localhost:9100/metrics` | Test from inside container |
| `docker port <container>` | Show port mappings |
| `docker inspect <container>` | View container details/IP |

## Quick Docker Compose Commands

| Command | Purpose |
|---------|---------|
| `docker-compose up -d` | Start services in background |
| `docker-compose ps` | Show status of services |
| `docker-compose logs` | View logs from all services |
| `docker-compose logs <service>` | View specific service logs |
| `docker-compose exec <service> bash` | Access service shell |
| `docker-compose restart` | Restart all services |
| `docker-compose down` | Stop and remove services |

## Metric Value Units (Common)

| Unit | Conversion | Example |
|------|-----------|---------|
| Bytes | ÷ 1024 = KB | `1048576` bytes = 1 MB |
| Seconds | For rate: ÷ duration | 60s counter / 60s = 1 per second |
| Unix Timestamp | For readable: multiply × 1000 | `1705329600` seconds |
| Percentage | Usually 0-100 | `45.3` = 45.3% |
| Nanoseconds | ÷ 1e9 = seconds | `500000000` ns = 0.5s |

## Quick Health Check Script

```bash
#!/bin/bash
echo "1. Prometheus status:"
curl -s http://localhost:9090/-/healthy || echo "DOWN"

echo -e "\n2. Node Exporter status:"
curl -s http://localhost:9100/metrics | head -1 || echo "DOWN"

echo -e "\n3. Check targets:"
curl -s http://localhost:9090/api/v1/targets | jq '.data.activeTargets[] | {job, state}' || echo "ERROR"

echo -e "\n4. Sample query:"
curl -s -G 'http://localhost:9090/api/v1/query' --data-urlencode 'query=up' | jq '.data.result[]' || echo "ERROR"
```

---

**Keep this handy while working with exporters!**
