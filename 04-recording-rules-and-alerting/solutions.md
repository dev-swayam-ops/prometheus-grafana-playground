# Solutions: Recording Rules and Alerting

## Solution 1: Create Simple Recording Rule
**In rules.yml**:
```yaml
groups:
  - name: memory_rules
    interval: 15s
    rules:
      - record: node:memory_available:pct
        expr: (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100
```

**Update prometheus.yml**:
```yaml
rule_files:
  - /etc/prometheus/rules.yml
```

**Explanation**: 
- `record` creates new metric name
- Expression runs every 15s (interval)
- Result stored as `node:memory_available:pct`
- Naming convention: `<level>:<metric>:<aggregation><period>`

**Query in UI**: `node:memory_available:pct`

**Expected Output**:
```
node:memory_available:pct 45.3
```

---

## Solution 2: Access Prometheus Rules UI
**Steps**:
1. Open browser: `http://localhost:9090/rules`
2. See rule groups, rules, and evaluation status
3. Click rule to see details

**Explanation**: 
- Rules page shows all loaded rules
- Green = last evaluation OK
- Red = last evaluation failed
- Shows evaluation interval and next evaluation time

**Expected**: Recording rules and alert rules listed with evaluation results

---

## Solution 3: Write Basic Alert Rule
**In rules.yml**:
```yaml
groups:
  - name: memory_alerts
    interval: 15s
    rules:
      - alert: HighMemoryUsage
        expr: node:memory_available:pct < 10
        for: 1m
        labels:
          severity: warning
        annotations:
          summary: "Memory available below 10%"
```

**Explanation**: 
- `alert` defines alert name
- `expr` is PromQL condition
- `for: 1m` means condition must be true for 1 minute before firing
- `labels` and `annotations` provide context

**Verification**: View in Prometheus Alerts tab

---

## Solution 4: Test Alert Expression
**Steps**:
1. Open: `http://localhost:9090/graph`
2. Query: `node_memory_MemAvailable_bytes < 1000000000`
3. Click "Execute"

**Explanation**: 
- Always test PromQL before using in alerts
- Graph tab shows instant vector results
- Table tab shows values
- If metric doesn't exist, alert will never fire

**Expected Output**:
```
node_memory_MemAvailable_bytes{instance="...", job="node"} 8589934592
```
(Or empty if condition not met)

---

## Solution 5: View Alert Status
**Steps**:
1. Open: `http://localhost:9090/alerts`
2. View alert list:
   - **Inactive**: Condition not met
   - **Pending**: Condition true, waiting for "for" duration
   - **Firing**: Duration reached, alert active

**Explanation**: 
- Alert state transitions: Inactive → Pending → Firing
- Minimum 1 evaluation cycle before state change
- Remains pending for configured "for" duration

**Expected**:
```
HighMemoryUsage [INACTIVE]
TargetDown [FIRING] for 2m
```

---

## Solution 6: Query Alert Metrics
**Query**:
```promql
ALERTS{alertname="HighMemoryUsage"}
```

**Alternative**:
```promql
count(ALERTS{alertname="HighMemoryUsage", alertstate="firing"})
```

**Explanation**: 
- ALERTS is built-in metric showing alert status
- Value: 1 if firing, not in result if inactive
- `alertstate` label shows "firing" or "pending"
- Useful for counting/tracking alerts

**Expected Output**:
```
ALERTS{alertname="HighMemoryUsage", alertstate="firing"} 1
```
(Or no result if inactive)

---

## Solution 7: Recording Rule with Labels
**In rules.yml**:
```yaml
  - name: network_rules
    interval: 15s
    rules:
      - record: job:network_rate:5m
        expr: sum(rate(node_network_receive_bytes_total[5m])) by (job)
```

**Query**: `job:network_rate:5m`

**Explanation**: 
- `by (job)` groups result by job label
- Result has separate time-series per job
- Useful for per-service metrics
- `without (label)` excludes label instead

**Expected Output**:
```
job:network_rate:5m{job="node"} 12345.67
job:network_rate:5m{job="prometheus"} 98.76
```

---

## Solution 8: Alert with Custom Labels
**In rules.yml**:
```yaml
  - alert: CustomAlert
    expr: up == 0
    for: 2m
    labels:
      severity: critical
      team: devops
      env: prod
```

**Explanation**: 
- Custom labels used for routing in AlertManager
- Any label can be added
- Labels are sent with alert to AlertManager
- Used in `match` conditions for routing

**Verification**: 
- View in Prometheus Alerts tab
- Labels shown for alert

---

## Solution 9: Alert with Annotations
**In rules.yml**:
```yaml
    annotations:
      summary: "Target down: {{ $labels.instance }}"
      description: "Target has been down for {{ $value }}s"
      runbook: "https://wiki.example.com/runbooks"
```

**Template Variables**:
- `{{ $labels.instance }}` - Label value
- `{{ $value }}` - Metric value
- `{{ $timestamp }}` - Timestamp

**Explanation**: 
- Annotations are metadata (not used for routing)
- Summary is brief (shown in UI/emails)
- Description provides details
- Runbook links to documentation

**Verification**: View in AlertManager notifications

---

## Solution 10: Check Rule Evaluation Logs
**Command**:
```bash
docker logs prometheus | grep -i "rule\|alert" | tail -20
```

**Alternative - Check config loaded**:
```bash
docker logs prometheus | grep "rule files"
```

**Alternative - Real-time logs**:
```bash
docker logs -f prometheus | grep -i "alert"
```

**Explanation**: 
- Logs show when rules are loaded
- Shows evaluation errors
- Shows alertmanager communication
- Useful for debugging failed rules

**Expected Output**:
```
level=info ts=2024-01-15 msg="Loading rule files" files=rules.yml
level=info ts=2024-01-15 msg="Sending alerts to AlertManager"
```

---

**All Solutions Complete ✅**
