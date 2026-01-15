# Cheatsheet: Service Discovery and Scraping

## Scrape Configuration

| Parameter | Default | Purpose |
|-----------|---------|---------|
| `scrape_interval` | 15s | How often to scrape targets |
| `scrape_timeout` | 10s | Request timeout |
| `metrics_path` | /metrics | HTTP path for metrics |
| `scheme` | http | http or https |
| `honor_labels` | false | Keep target labels over job labels |
| `honor_timestamps` | true | Use metric timestamps |

## Service Discovery Types

| Type | Config | Use Case |
|------|--------|----------|
| **Static** | `static_configs` | Fixed list of targets |
| **File-based** | `file_sd_configs` | Dynamic JSON/YAML file |
| **DNS** | `dns_sd_configs` | DNS A/SRV records |
| **Kubernetes** | `kubernetes_sd_configs` | K8s pods/services |
| **Consul** | `consul_sd_configs` | Consul catalog |
| **AWS** | `ec2_sd_configs` | EC2 instances |

## Common Commands

| Command | Purpose | Example |
|---------|---------|---------|
| **View targets** | Open targets page | `http://localhost:9090/targets` |
| **Query targets API** | Get target list | `curl http://localhost:9090/api/v1/targets` |
| **Reload config** | Apply config changes | `curl -X POST http://localhost:9090/-/reload` |
| **View active targets** | Filter by state | `curl 'http://localhost:9090/api/v1/targets?state=active'` |
| **View dropped targets** | Targets filtered out | `curl 'http://localhost:9090/api/v1/targets?state=dropped'` |
| **Check config** | View loaded config | `curl http://localhost:9090/config` |

## Static Targets Configuration

```yaml
scrape_configs:
  - job_name: 'my-job'
    scrape_interval: 15s
    scrape_timeout: 10s
    metrics_path: '/metrics'
    scheme: 'http'
    static_configs:
      - targets: ['localhost:9100', 'localhost:9101']
        labels:
          datacenter: us-east
          env: production
```

## File-based Discovery

**prometheus.yml**:
```yaml
scrape_configs:
  - job_name: 'file-sd'
    file_sd_configs:
      - files:
        - '/etc/prometheus/targets/*.json'
        refresh_interval: 5s
```

**targets.json**:
```json
[
  {
    "targets": ["host1:9100", "host2:9100"],
    "labels": {
      "env": "prod",
      "datacenter": "us-east"
    }
  },
  {
    "targets": ["host3:9100"],
    "labels": {
      "env": "staging"
    }
  }
]
```

## Relabeling

| Action | Purpose | Example |
|--------|---------|---------|
| **replace** | Replace label values | Rename labels |
| **keep** | Keep matching targets | Filter targets |
| **drop** | Remove matching targets | Exclude targets |
| **labeldrop** | Delete labels | Remove metadata |
| **labelkeep** | Keep specific labels | Remove unwanted labels |
| **hashmod** | Hash-based assignment | Load balancing |

**Keep only specific targets**:
```yaml
relabel_configs:
  - source_labels: [__address__]
    regex: 'localhost:9100'
    action: keep
```

**Drop based on label**:
```yaml
relabel_configs:
  - source_labels: [__meta_kubernetes_pod_label_app]
    regex: 'ignore-.*'
    action: drop
```

**Extract hostname from address**:
```yaml
relabel_configs:
  - source_labels: [__address__]
    regex: '([^:]+):.*'
    replacement: '${1}'
    target_label: hostname
```

## Special Labels (Service Discovery)

| Label | Meaning | Scope |
|-------|---------|-------|
| `__address__` | Target host:port | All |
| `__scheme__` | http or https | All |
| `__metrics_path__` | Metrics path | All |
| `__param_<name>` | URL parameter | All |
| `__meta_consul_*` | Consul metadata | Consul-SD |
| `__meta_kubernetes_*` | K8s metadata | Kubernetes-SD |
| `__meta_aws_*` | AWS metadata | EC2-SD |

## Relabel Configuration Example

```yaml
scrape_configs:
  - job_name: 'my-service'
    static_configs:
      - targets: ['host1:9100', 'host2:9100']
    relabel_configs:
      # Keep only host1
      - source_labels: [__address__]
        regex: 'host1:.*'
        action: keep
      
      # Rename __address__ to server
      - source_labels: [__address__]
        target_label: server
      
      # Add custom label
      - target_label: env
        replacement: 'production'
```

## DNS-based Discovery

```yaml
scrape_configs:
  - job_name: 'dns-sd'
    dns_sd_configs:
      - names: ['example.com']
        refresh_interval: 30s
        port: 9100
```

## Troubleshooting

| Issue | Check |
|-------|-------|
| Targets DOWN | Verify exporter running: `curl http://target:9100/metrics` |
| Targets not appearing | Check relabel action: keep/drop |
| Labels missing | Verify static_configs labels or relabel_configs |
| Metrics not scraped | Check metrics_path and scheme match exporter |
| File-SD not updating | Check file path and refresh_interval |
| Reload fails | Validate YAML syntax: `prometheus --config.file prometheus.yml` |

## Quick Setup

**Basic static config**:
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
        labels:
          datacenter: us-east
```

**With file-based discovery**:
```yaml
scrape_configs:
  - job_name: 'dynamic'
    file_sd_configs:
      - files: ['targets.json']
    relabel_configs:
      - source_labels: [__address__]
        regex: 'prod-.*'
        action: keep
```

## Best Practices

✅ **DO**:
- Test target URL before adding: `curl http://target:9100/metrics`
- Use meaningful labels for organization
- Start with longer scrape intervals (60s) for stability
- Use relabel_configs to filter instead of manual management
- Document custom metrics paths and non-standard ports
- Keep relabel logic simple and readable

❌ **DON'T**:
- Use very short scrape intervals < 5s without reason
- Forget to reload after config changes
- Mix static and dynamic targets in same job
- Use overly complex relabel regex
- Leave service discovery unconfigured
- Scrape all labels if you only need a few

---

**Keep this handy for service discovery setup!**
