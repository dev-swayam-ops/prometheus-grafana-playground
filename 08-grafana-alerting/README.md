# 08 - Grafana Alerting

## What You'll Learn
- Creating alerts directly in Grafana (vs Prometheus)
- Alert rule conditions and evaluation
- Notification policies and routing
- Integration with AlertManager
- Alert testing and validation
- Using alert states and transitions
- Best practices for Grafana alerts

## Prerequisites
- [07-grafana-dashboards-and-panels](../07-grafana-dashboards-and-panels/README.md) completed
- Grafana running with data source configured
- AlertManager running (optional, for routing)
- Understanding of alert conditions and PromQL
- Docker and Docker Compose installed

## Key Concepts

### Grafana Alerts vs Prometheus Alerts

| Feature | Grafana Alerts | Prometheus Alerts |
|---------|---|---|
| Configuration | Web UI or JSON | YAML files |
| Evaluation | In Grafana | In Prometheus |
| Data sources | Any data source | Prometheus only |
| Routing | Limited | Full AlertManager |
| Latency | Depends on evaluation interval | Standard |

### Alert Components

```
Alert Rule → Condition → State → Notification
         ↓
    Evaluation interval
    (e.g., every 1 minute)
```

**Alert States**:
- **Normal**: Condition not met
- **Pending**: Condition true, waiting for duration
- **Alerting**: Duration reached, alert firing
- **No Data**: No data available to evaluate

### Alert Policies

```yaml
Routing:
  Default receiver: "default"
  Rules:
    - Match condition → Route to receiver
    - Send notification
    - Set duration/repeat
```

## Hands-on Lab: Create Grafana Alert Rule

### Step 1: Open Dashboard and Panel
1. Open Grafana: `http://localhost:3000`
2. Open any dashboard with metric data
3. Click on a panel (e.g., CPU Usage)
4. Click gear icon → "Edit"

### Step 2: Go to Alert Rules
1. Click "Alert" tab in panel editor
2. Click "Create alert rule"
3. Set rule name: "High CPU Usage"

### Step 3: Set Alert Condition
```
Condition: "when avg() of query (A, last 5m) is above 80"
```

**Configuration**:
```
Query: node_cpu_seconds_total metric
Reducer: avg
Threshold: 80 (percent)
Evaluation interval: 1m
For duration: 5m
```

### Step 4: Add Alert Labels
```yaml
Labels:
  severity: critical
  team: devops
  environment: prod
```

### Step 5: Add Annotations
```yaml
Annotations:
  Summary: "High CPU usage detected"
  Description: "CPU is {{ $value }}% on {{ $labels.instance }}"
  Runbook: "https://wiki.example.com/high-cpu"
```

### Step 6: Configure Notification Channel
1. Click "Notification policy"
2. Select receiver: "default" or "AlertManager"
3. Set repeat interval: 4h

### Step 7: Save Alert Rule
1. Click "Save" button
2. Confirm: Alert rule created message

**Expected Output**:
```
Alert rule "High CPU Usage" created successfully
Status: Normal (no alert firing)
```

### Step 8: Test Alert Rule
To force alert state change:
1. Lower threshold in condition to current value
2. Save and observe state change
3. Reset threshold back to original

**Expected**: Alert transitions: Normal → Pending → Alerting

## Validation

Verify alert setup:
```bash
# Check Grafana alerts API
curl -H "Authorization: Bearer TOKEN" \
  http://localhost:3000/api/ruler/grafana/rules

# Check alert status
curl http://localhost:3000/api/ruler/grafana/rules/default

# View alert instances
curl -H "Authorization: Bearer TOKEN" \
  http://localhost:3000/api/v1/rules
```

## Cleanup

To delete alert rule:
1. Open alert rule in Grafana
2. Click "Delete" button
3. Confirm deletion

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| "No Data" alert always firing | Add "If no data" option: "Set state to No Data" |
| Alert never fires | Check threshold and data actually reaches it |
| Too many notifications | Increase repeat_interval or group_interval |
| Alert stuck in pending | Reduce "For" duration or lower threshold |
| Can't find alert rules | Check correct rule folder/namespace |

## Troubleshooting

**Issue**: Alert rule not evaluating
```bash
# Check Grafana logs
docker logs grafana | grep -i alert

# Check rule configuration
curl -H "Authorization: Bearer TOKEN" \
  http://localhost:3000/api/ruler/grafana/rules | jq '.'
```

**Issue**: Notifications not being sent
```bash
# Check if AlertManager is connected
curl http://localhost:3000/api/ruler/grafana/rules/default | jq '.alertmanager'

# Check notification policy
curl -H "Authorization: Bearer TOKEN" \
  http://localhost:3000/api/v1/provisioning/policies
```

**Issue**: Alert threshold not triggering
```bash
# Test metric in Prometheus
curl -G 'http://localhost:9090/api/v1/query' \
  --data-urlencode 'query=your_metric'

# Verify actual values exceed threshold
```

## Next Steps

✅ Grafana alerting configured!

**Ready to proceed to**:
- [09-kubernetes-monitoring](../09-kubernetes-monitoring/README.md) - Monitor Kubernetes clusters
- [exercises](./exercises.md) - Practice with hands-on exercises
