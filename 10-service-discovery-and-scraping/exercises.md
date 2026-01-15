# Exercises: Service Discovery and Scraping

## Exercise 1: View Current Prometheus Targets (Easy)
Access Prometheus targets page to see discovered targets.

**Task**:
1. Open `http://localhost:9090`
2. Click "Status" → "Targets"
3. Observe list of targets, their status, and labels

**Validation**: Targets page shows UP/DOWN status.

---

## Exercise 2: Add Static Target (Easy)
Add a new static target to prometheus.yml.

**Task**:
1. Edit prometheus.yml
2. Add new scrape_config:
```yaml
- job_name: 'my-service'
  static_configs:
    - targets: ['localhost:8080']
```
3. Save and reload
4. Check targets page

**Validation**: New target appears in targets list.

---

## Exercise 3: Set Scrape Interval (Easy)
Configure scrape frequency for a job.

**Task**:
```yaml
scrape_configs:
  - job_name: 'fast-metrics'
    scrape_interval: 5s    # Faster collection
    scrape_timeout: 3s
    static_configs:
      - targets: ['localhost:9100']
```

**Validation**: Job scrapes every 5 seconds (check Prometheus graph).

---

## Exercise 4: Use File-based Discovery (Easy)
Load targets from external JSON file.

**Task**:
1. Create targets.json:
```json
[
  {
    "targets": ["localhost:9100"],
    "labels": {"env": "prod"}
  }
]
```
2. Add to prometheus.yml:
```yaml
file_sd_configs:
  - files: ['/path/to/targets.json']
```
3. Check targets page

**Validation**: Targets from file appear in targets list.

---

## Exercise 5: Add Target Labels (Medium)
Assign custom labels to targets.

**Task**:
```yaml
static_configs:
  - targets: ['localhost:9100', 'localhost:9101']
    labels:
      datacenter: us-east
      service: monitoring
```

**Validation**: Labels visible in targets page and queries.

---

## Exercise 6: Query Target Metadata (Medium)
Use Prometheus API to list all targets.

**Task**:
```bash
curl 'http://localhost:9090/api/v1/targets' | jq '.data.activeTargets'
```

**Validation**: Returns JSON with target addresses, labels, status.

---

## Exercise 7: Keep Only Specific Targets (Medium)
Use relabeling to filter targets.

**Task**:
Add relabel_configs:
```yaml
relabel_configs:
  - source_labels: [__address__]
    regex: 'localhost:9100'
    action: keep
```

**Validation**: Only matching targets remain active.

---

## Exercise 8: Rename Target Label (Medium)
Relabel __address__ to custom label.

**Task**:
```yaml
relabel_configs:
  - source_labels: [__address__]
    target_label: instance_address
```

**Validation**: New label appears in target labels.

---

## Exercise 9: Set Custom Metrics Path (Medium)
Specify custom metrics endpoint path.

**Task**:
```yaml
- job_name: 'custom-app'
  metrics_path: '/prometheus/metrics'
  static_configs:
    - targets: ['localhost:8080']
```

**Validation**: Prometheus scrapes from custom path, no error 404.

---

## Exercise 10: Handle Dynamic Target List (Medium)
Reload targets.json without restarting Prometheus.

**Task**:
1. Create targets.json with initial targets
2. Add to prometheus.yml with file_sd
3. Update targets.json (add/remove entry)
4. Reload: `curl -X POST http://localhost:9090/-/reload`
5. Check targets page

**Validation**: New targets appear/old targets disappear without restart.

---

**All Exercises Complete ✅**
