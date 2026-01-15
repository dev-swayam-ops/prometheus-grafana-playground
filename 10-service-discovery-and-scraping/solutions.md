# Solutions: Service Discovery and Scraping

## Solution 1: View Current Prometheus Targets

**Steps**:
1. Open browser: `http://localhost:9090`
2. Click "Status" → "Targets"
3. Scroll to see all targets

**Explanation**: 
- Targets page shows all configured scrape targets
- Status shows UP (green) or DOWN (red)
- Labels column shows applied labels
- Last Scrape shows timestamp of last collection

**Expected**: Targets list with status indicators

---

## Solution 2: Add Static Target

**Edit prometheus.yml**:
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'my-service'
    static_configs:
      - targets: ['localhost:8080']
```

**Reload**:
```bash
curl -X POST http://localhost:9090/-/reload
```

**Verification**:
```bash
curl 'http://localhost:9090/api/v1/targets?job=my-service' | jq
```

**Explanation**: 
- Static targets are defined directly in config
- Reload applies changes without restart
- New target appears within scrape_interval

---

## Solution 3: Set Scrape Interval

**Configuration**:
```yaml
scrape_configs:
  - job_name: 'fast-metrics'
    scrape_interval: 5s
    scrape_timeout: 3s
    static_configs:
      - targets: ['localhost:9100']
```

**Explanation**: 
- scrape_interval: How often to pull metrics (default 15s)
- scrape_timeout: Max wait for response (must be < scrape_interval)
- Shorter intervals = more frequent updates, more load
- Default 15s is usually fine

**Expected**: Metrics updated every 5 seconds

---

## Solution 4: Use File-based Discovery

**Create targets.json**:
```json
[
  {
    "targets": ["localhost:9100"],
    "labels": {"env": "prod"}
  },
  {
    "targets": ["localhost:9101"],
    "labels": {"env": "staging"}
  }
]
```

**Update prometheus.yml**:
```yaml
scrape_configs:
  - job_name: 'file-sd'
    file_sd_configs:
      - files: ['/etc/prometheus/targets.json']
        refresh_interval: 5s
```

**Reload**:
```bash
curl -X POST http://localhost:9090/-/reload
```

**Explanation**: 
- File-based discovery loads targets from JSON file
- Prometheus periodically checks file for updates
- Labels from file are applied to targets
- Useful for dynamic infrastructure

---

## Solution 5: Add Target Labels

**Configuration**:
```yaml
scrape_configs:
  - job_name: 'my-job'
    static_configs:
      - targets: ['localhost:9100']
        labels:
          datacenter: us-east
          service: monitoring
          team: platform
```

**Reload**:
```bash
curl -X POST http://localhost:9090/-/reload
```

**Verify Labels**:
```bash
curl 'http://localhost:9090/api/v1/targets' | jq '.data.activeTargets[0].labels'
```

**Output**:
```json
{
  "datacenter": "us-east",
  "service": "monitoring",
  "team": "platform",
  "job": "my-job",
  "instance": "localhost:9100"
}
```

**Explanation**: 
- Labels added in static_configs are applied to all targets
- Labels help organize and filter targets
- Appear in all metrics from that target

---

## Solution 6: Query Target Metadata

**Command**:
```bash
curl 'http://localhost:9090/api/v1/targets' | jq '.data.activeTargets[] | {address: .labels.instance, job: .labels.job, health: .health}'
```

**Output**:
```json
{
  "address": "localhost:9100",
  "job": "node-exporter",
  "health": "up"
}
```

**Get Dropped Targets**:
```bash
curl 'http://localhost:9090/api/v1/targets?state=dropped' | jq
```

**Explanation**: 
- API endpoint shows active and dropped targets
- Dropped targets excluded by relabel action: drop
- Useful for debugging discovery issues

---

## Solution 7: Keep Only Specific Targets

**Configuration**:
```yaml
scrape_configs:
  - job_name: 'filtered'
    static_configs:
      - targets: ['localhost:9100', 'localhost:9101', 'localhost:9102']
    relabel_configs:
      - source_labels: [__address__]
        regex: 'localhost:9100'
        action: keep
```

**Actions**:
- `keep` - Keep matching targets only
- `drop` - Remove matching targets
- `replace` - Replace label values

**Reload**:
```bash
curl -X POST http://localhost:9090/-/reload
```

**Verification**: Only localhost:9100 remains UP.

**Explanation**: 
- Relabeling filters targets before scraping
- Reduces noise for unwanted targets
- Useful for multi-environment setups

---

## Solution 8: Rename Target Label

**Configuration**:
```yaml
relabel_configs:
  - source_labels: [__address__]
    target_label: instance_address
    replacement: '${1}'
```

**Alternative - Extract port**:
```yaml
relabel_configs:
  - source_labels: [__address__]
    regex: '(.+):(.+)'
    replacement: '${2}'
    target_label: port
```

**Verification**:
```bash
curl 'http://localhost:9090/api/v1/targets' | jq '.data.activeTargets[0].labels'
```

**Explanation**: 
- Relabeling creates new labels from existing ones
- `${1}, ${2}` reference regex groups
- Useful for structured label extraction

---

## Solution 9: Set Custom Metrics Path

**Configuration**:
```yaml
scrape_configs:
  - job_name: 'custom-app'
    metrics_path: '/prometheus/metrics'
    scheme: 'https'
    static_configs:
      - targets: ['api.example.com:443']
```

**Test Endpoint**:
```bash
curl http://localhost:8080/prometheus/metrics
```

**Reload**:
```bash
curl -X POST http://localhost:9090/-/reload
```

**Explanation**: 
- metrics_path: Custom HTTP path (default: /metrics)
- scheme: Protocol (http or https)
- Useful for apps with non-standard metrics paths
- Test endpoint manually first

---

## Solution 10: Handle Dynamic Target List

**Initial targets.json**:
```json
[
  {"targets": ["localhost:9100"], "labels": {"env": "prod"}}
]
```

**Add to prometheus.yml**:
```yaml
file_sd_configs:
  - files: ['/etc/prometheus/targets.json']
    refresh_interval: 5s
```

**Update targets.json** (add new target):
```json
[
  {"targets": ["localhost:9100"], "labels": {"env": "prod"}},
  {"targets": ["localhost:9101"], "labels": {"env": "staging"}}
]
```

**Reload**:
```bash
curl -X POST http://localhost:9090/-/reload
```

**Verification**:
```bash
curl 'http://localhost:9090/api/v1/targets' | jq '.data.activeTargets | length'
```

**Explanation**: 
- File-SD checks for file changes periodically
- reload endpoint refreshes discovery
- No downtime when adding/removing targets
- Perfect for dynamic infrastructure

---

**All Solutions Complete ✅**
