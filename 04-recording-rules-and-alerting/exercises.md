# Exercises: Recording Rules and Alerting

## Exercise 1: Create Simple Recording Rule (Easy)
Create a recording rule that calculates average memory usage.

**Task**:
In `rules.yml`:
```yaml
groups:
  - name: memory_rules
    rules:
      - record: node:memory_available:pct
        expr: (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100
```

Update prometheus.yml: `rule_files: - /etc/prometheus/rules.yml`

Reload Prometheus and query: `node:memory_available:pct`

**Validation**: Recording rule returns percentage 0-100.

---

## Exercise 2: Access Prometheus Rules UI (Easy)
View all loaded rules in Prometheus.

**Task**:
1. Open: `http://localhost:9090/rules`
2. Observe: Recording rules and alert rules listed
3. Check rule evaluation interval

**Validation**: Rules page shows rule groups and evaluation results.

---

## Exercise 3: Write Basic Alert Rule (Easy)
Create alert for high memory usage.

**Task**:
In `rules.yml`:
```yaml
  - name: memory_alerts
    rules:
      - alert: HighMemoryUsage
        expr: node:memory_available:pct < 10
        for: 1m
        labels:
          severity: warning
```

**Validation**: Alert appears in Prometheus Alerts tab.

---

## Exercise 4: Test Alert Expression (Easy)
Test alert PromQL expression before creating rule.

**Task**:
1. Open Prometheus Graph tab
2. Enter alert expression: `node_memory_MemAvailable_bytes < 1000000000`
3. Click Execute

**Validation**: Expression returns current metric values.

---

## Exercise 5: View Alert Status (Medium)
Check alert states (inactive, pending, firing).

**Task**:
1. Open: `http://localhost:9090/alerts`
2. Click alert to see:
   - Alert name
   - State (Inactive/Pending/Firing)
   - Value
   - For duration

**Validation**: Alerts tab shows all configured alerts and their states.

---

## Exercise 6: Query Alert Metrics (Medium)
Query ALERTS metric showing alert status.

**Task**:
In Prometheus Graph tab:
```promql
ALERTS{alertname="HighMemoryUsage"}
```

**Validation**: Returns 1 if firing, 0 if inactive.

---

## Exercise 7: Configure Recording Rule with Labels (Medium)
Create recording rule that groups by job.

**Task**:
```yaml
  - record: job:network_rate:5m
    expr: sum(rate(node_network_receive_bytes_total[5m])) by (job)
```

Query: `job:network_rate:5m`

**Validation**: Returns separate value for each job.

---

## Exercise 8: Set Alert with Custom Labels (Medium)
Add custom labels to alert for routing.

**Task**:
```yaml
  - alert: CustomAlert
    expr: up == 0
    for: 2m
    labels:
      severity: critical
      team: devops
      env: prod
```

**Validation**: Alert shows custom labels in UI.

---

## Exercise 9: View Alert Annotations (Medium)
Add description and summary to alert.

**Task**:
```yaml
    annotations:
      summary: "Target down: {{ $labels.instance }}"
      description: "Target has been down for {{ $value }}s"
      runbook: "https://wiki.example.com/runbooks"
```

View in UI: Alerts tab shows annotations.

**Validation**: Annotations display in AlertManager.

---

## Exercise 10: Check Rule Evaluation Logs (Medium)
View rule evaluation details in Prometheus logs.

**Task**:
```bash
docker logs prometheus | grep -i "rule\|alert" | tail -20
```

**Validation**: Logs show rule evaluation status and errors.

---

**All Exercises Complete ✅**
