# Service Discovery and Scraping

## What You'll Learn
- Prometheus service discovery mechanisms (DNS, static, file-based, Kubernetes)
- How Prometheus scrapes targets and collects metrics
- Scrape configuration (intervals, timeouts, paths)
- Target relabeling for dynamic target management
- Service discovery in different environments

## Prerequisites
- Module 01: Prometheus Basics (understanding Prometheus)
- Module 02: Metrics and Exporters (exporter concepts)
- Docker and docker-compose
- Basic YAML knowledge

## Key Concepts

### Service Discovery Types
1. **Static targets** - Fixed list in prometheus.yml
2. **DNS-SD** - Discover targets via DNS records
3. **File-SD** - Dynamic target list from JSON/YAML files
4. **Kubernetes-SD** - Discover pods and services in Kubernetes
5. **Consul-SD** - Service discovery from Consul
6. **Cloud provider SD** - AWS, GCP, Azure service discovery

### Scrape Configuration
```yaml
scrape_configs:
  - job_name: 'my-job'
    scrape_interval: 30s      # How often to scrape
    scrape_timeout: 10s       # Request timeout
    metrics_path: '/metrics'  # Where to fetch metrics
    static_configs:
      - targets:
        - 'localhost:9090'
```

### Relabeling
Process of modifying labels before ingestion:
```yaml
relabel_configs:
  - source_labels: [__address__]
    target_label: instance
    replacement: 'my-server'
```

### Service Discovery Labels
- `__address__` - Target address (host:port)
- `__scheme__` - HTTP or HTTPS
- `__metrics_path__` - Default /metrics
- `__meta_*` - Discovery-specific metadata

## Hands-on Lab

**Objective**: Configure Prometheus with multiple service discovery methods.

### Step 1: Create docker-compose.yml
```yaml
version: '3.8'
services:
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./targets.json:/etc/prometheus/targets.json
      - prometheus_data:/prometheus
    ports:
      - "9090:3000"
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'

  node-exporter-1:
    image: prom/node-exporter:latest
    ports:
      - "9100:9100"

  node-exporter-2:
    image: prom/node-exporter:latest
    ports:
      - "9101:9100"

volumes:
  prometheus_data:
```

### Step 2: Create prometheus.yml
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  # Static targets
  - job_name: 'node-static'
    static_configs:
      - targets: ['localhost:9100', 'localhost:9101']
        labels:
          group: 'production'

  # File-based discovery
  - job_name: 'node-file'
    file_sd_configs:
      - files:
        - '/etc/prometheus/targets.json'

  # Prometheus itself
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

### Step 3: Create targets.json
```json
[
  {
    "targets": ["localhost:9100"],
    "labels": {
      "job": "node-file",
      "datacenter": "us-east"
    }
  },
  {
    "targets": ["localhost:9101"],
    "labels": {
      "job": "node-file",
      "datacenter": "us-west"
    }
  }
]
```

### Step 4: Start services
```bash
docker-compose up -d
sleep 10
curl http://localhost:9090/targets
```

**Expected Output**: 
```
Status: UP (all targets healthy)
- node-static job: 2 targets UP
- node-file job: 2 targets UP
- prometheus job: 1 target UP
```

### Step 5: Query discovered targets
```bash
# Check all discovered targets
curl 'http://localhost:9090/api/v1/targets' | jq '.data.activeTargets'
```

**Expected Output**: Shows target address, labels, and status.

### Step 6: Test relabeling
Edit prometheus.yml, add relabel config:
```yaml
relabel_configs:
  - source_labels: [__address__]
    regex: 'localhost:9100'
    action: keep
```

Reload: `curl -X POST http://localhost:9090/-/reload`

**Expected**: Only localhost:9100 target remains.

## Validation
✅ Access `http://localhost:9090/targets` - should show UP targets
✅ Run query: `up` - should return value 1 for healthy targets
✅ View target labels: `curl http://localhost:9090/api/v1/targets`
✅ Relabeling works: targets filtered correctly

## Cleanup
```bash
docker-compose down
docker volume rm service-discovery-and-scraping_prometheus_data
```

## Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| Invalid YAML syntax | Service won't start | Validate YAML, check indentation |
| Wrong metrics_path | No metrics collected | Default is /metrics, verify exporter path |
| Timeout too short | Targets marked DOWN | Increase scrape_timeout |
| Missing relabel action | Labels not applied | Specify action: replace/drop/keep |
| File-SD file not found | Job fails | Check file path and permissions |

## Troubleshooting

**Targets showing DOWN**:
```bash
curl http://localhost:9100/metrics  # Check exporter is running
curl http://localhost:9090/config   # View actual Prometheus config
```

**Service discovery not working**:
```bash
# Check Prometheus logs
docker logs prometheus

# Verify SD config
curl http://localhost:9090/api/v1/targets?state=any
```

**Labels not applied**:
- Check relabel_configs syntax
- Reload with: `curl -X POST http://localhost:9090/-/reload`
- View targets page to see final labels

## Next Steps
- Module 11: SLO/SLI and Error Budgets (reliability metrics)
- Module 04: Recording Rules and Alerting (use discovered targets in rules)
- Advanced: Kubernetes service discovery, Consul integration
