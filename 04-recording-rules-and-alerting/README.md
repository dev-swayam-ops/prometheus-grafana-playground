# 04 - Recording Rules and Alerting

## What You'll Learn
- What are recording rules and why they're needed
- Creating recording rules in Prometheus
- Configuring alert rules (alert conditions)
- Alert rule syntax and evaluation
- Best practices for alerting
- Testing alert rules
- Understanding alert states: firing, pending, inactive

## Prerequisites
- [03-promql-queries](../03-promql-queries/README.md) completed
- Understanding of PromQL queries
- Running Prometheus with metrics available
- Docker and Docker Compose installed

## Key Concepts

### Recording Rules
Recording rules are pre-computed PromQL queries that:
1. Run on a schedule (every evaluation interval)
2. Store results as new time-series
3. Simplify expensive queries
4. Improve dashboard performance

**Benefits**:
- Reduces query complexity
- Improves dashboard performance
- Makes alerts more readable

### Alert Rules
Alert rules define conditions that trigger alerts:
1. Evaluate PromQL expression
2. If condition met for duration → "firing"
3. Send to AlertManager
4. AlertManager handles routing/notifications

**Alert States**:
```
Inactive → Pending (condition met) → Firing (duration reached) → Resolved
```

### Rule Syntax

```yaml
groups:
  - name: example_rules
    interval: 15s
    rules:
      # Recording rule
      - record: job:http_requests:rate5m
        expr: sum(rate(http_requests_total[5m])) by (job)

      # Alert rule
      - alert: HighErrorRate
        expr: rate(errors_total[5m]) > 0.05
        for: 5m  # Duration before firing
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value }}"
```

## Hands-on Lab: Create Recording and Alert Rules

### Step 1: Create rules file (rules.yml)
```yaml
global:
  evaluation_interval: 15s

groups:
  - name: cpu_rules
    interval: 15s
    rules:
      # Recording rule
      - record: node:cpu_idle:avg5m
        expr: avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) by (job)

  - name: alerts
    interval: 15s
    rules:
      # Alert rule 1
      - alert: HighCPUUsage
        expr: (1 - node:cpu_idle:avg5m) > 0.8
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on {{ $labels.job }}"
          description: "CPU idle is {{ $value | humanizePercentage }}"

      # Alert rule 2
      - alert: TargetDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Target {{ $labels.instance }} is down"
```

### Step 2: Update prometheus.yml
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - /etc/prometheus/rules.yml

alerting:
  alertmanagers:
    - static_configs:
        - targets: []

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']
```

### Step 3: Update docker-compose.yml
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
      - ./rules.yml:/etc/prometheus/rules.yml
    command:
      - --config.file=/etc/prometheus/prometheus.yml

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
```

### Step 4: Start Services
```bash
docker-compose up -d
docker-compose ps
```

**Expected Output**:
```
NAME              IMAGE                      STATUS
prometheus        prom/prometheus:latest     Up 5 seconds
node-exporter     prom/node-exporter:latest  Up 5 seconds
```

### Step 5: Check Rules Loaded
Open UI: `http://localhost:9090/rules`

**Expected**: Both recording and alert rules listed

### Step 6: View Recording Rule Results
In Prometheus UI (Graph tab):
```promql
node:cpu_idle:avg5m
```

**Expected Output**:
```
node:cpu_idle:avg5m{job="node"} 0.95
```

### Step 7: Check Alert Status
In Prometheus UI, click "Alerts" tab

**Expected**:
- Alert rules listed
- State: "inactive" or "firing" (depends on actual CPU)

### Step 8: Query Alert Metrics
```bash
curl -G 'http://localhost:9090/api/v1/query' \
  --data-urlencode 'query=ALERTS'
```

**Expected Output**: JSON with alert data

## Validation

Verify setup:
```bash
# Check rules loaded
curl http://localhost:9090/api/v1/rules | jq '.data.groups'

# Query recording rule result
curl -G 'http://localhost:9090/api/v1/query' \
  --data-urlencode 'query=node:cpu_idle:avg5m'

# Check alert status
curl http://localhost:9090/api/v1/alerts | jq '.data.alerts'
```

## Cleanup

```bash
docker-compose down
rm rules.yml prometheus.yml
```

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| Rules not loading | Check YAML syntax with: `yamllint rules.yml` |
| Recording rule not updating | Check evaluation interval vs scrape interval |
| Alert never fires | Verify expression in UI first (Graph tab) |
| Alert fires but no notification | AlertManager needed (see module 05) |
| Wrong timestamp in alerts | Use `time()` function for current time |

## Troubleshooting

**Issue**: Rules show error in UI
```bash
# Check logs
docker logs prometheus | grep -i "rule\|error"

# Validate YAML
docker run --rm -i sdesbonnes/yamllint < rules.yml
```

**Issue**: Recording rule shows no data
```promql
# Check if source metric exists
node_cpu_seconds_total

# Try simpler expression
avg(rate(node_cpu_seconds_total[5m]))
```

**Issue**: Alert stuck in pending state
```bash
# Check alert for duration
# Must remain true for "for" duration before firing

# View alert status
curl http://localhost:9090/api/v1/alerts | jq '.data.alerts[] | select(.labels.alertname=="YourAlert")'
```

## Next Steps

✅ Recording rules and alerting configured!

**Ready to proceed to**:
- [05-alertmanager-and-notifications](../05-alertmanager-and-notifications/README.md) - Route and send alerts
- [exercises](./exercises.md) - Practice with hands-on exercises
