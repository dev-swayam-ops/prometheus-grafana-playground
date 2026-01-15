# Cheatsheet: Grafana Alerting

## Alert Rule Components

| Component | Purpose | Example |
|-----------|---------|---------|
| **Name** | Rule identifier | "High CPU Usage" |
| **Condition** | Trigger criteria | "avg() > 80%" |
| **For** | Duration before firing | "5m", "10m" |
| **Labels** | Metadata for routing | `{severity: critical}` |
| **Annotations** | Human-readable text | Summary, Description |
| **Notification policy** | Route to receiver | AlertManager, webhook |

## Alert States

| State | Meaning | Next State |
|-------|---------|-----------|
| **Normal** | Condition not met | Pending (if true) |
| **Pending** | True for < "For" duration | Firing (if duration met) |
| **Firing** | Duration elapsed | Resolved (if false) |
| **NoData** | No data to evaluate | Normal/Pending |
| **Error** | Rule eval error | Normal (retry) |

## Alert Condition Syntax

```promql
# Simple threshold
avg(metric) > 80

# By dimension
sum(metric) by (instance) > 100

# Rate change
rate(counter[5m]) > 0.05

# Percentage
(metric1 / metric2) * 100 > 50
```

## API Endpoints for Alerts

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/ruler/grafana/rules` | GET | List all alert rules |
| `/api/ruler/grafana/rules` | POST | Create rule |
| `/api/ruler/grafana/rules/{uid}` | PUT | Update rule |
| `/api/ruler/grafana/rules/{uid}` | DELETE | Delete rule |
| `/api/v1/rules` | GET | List alert instances |
| `/api/v1/rules/{uid}` | GET | Get rule details |

## curl Commands

**Get all alert rules**:
```bash
curl -H "Authorization: Bearer TOKEN" \
  http://localhost:3000/api/ruler/grafana/rules
```

**Create alert rule**:
```bash
curl -X POST -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d @alert-rule.json \
  http://localhost:3000/api/ruler/grafana/rules
```

**List alert instances (current state)**:
```bash
curl -H "Authorization: Bearer TOKEN" \
  http://localhost:3000/api/v1/rules
```

## Annotation Template Variables

| Variable | Value | Example |
|----------|-------|---------|
| `{{ $value }}` | Metric value | 85.5 |
| `{{ $labels.instance }}` | Label value | "localhost:9100" |
| `{{ $labels.severity }}` | Custom label | "critical" |
| `{{ grafana_dashboard_url }}` | Dashboard link | http://grafana/d/... |
| `{{ grafana_panel_url }}` | Panel link | http://grafana/d/.../... |

## Notification Policy

```yaml
notification_policy:
  receiver: 'default'
  group_by:
    - alertname
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 4h
```

## Common Alert Conditions

| Scenario | Query |
|----------|-------|
| **High CPU** | `(1 - avg(rate(idle[5m]))) > 0.8` |
| **Low Memory** | `available / total < 0.1` |
| **High Disk** | `used / total > 0.9` |
| **Error Rate** | `errors / requests > 0.05` |
| **Queue Backing Up** | `increase(queue[5m]) > 0` |
| **Target Down** | `up == 0` |
| **Latency High** | `p95_latency > 1000ms` |

## Alert Rule JSON Structure

```json
{
  "uid": "alert-rule-uid",
  "title": "Alert Title",
  "condition": "B",
  "data": [
    {
      "refId": "A",
      "queryType": "metrics",
      "model": {
        "expr": "up"
      }
    }
  ],
  "for": "5m",
  "annotations": {
    "summary": "Alert summary",
    "description": "Alert description"
  },
  "labels": {
    "severity": "critical"
  }
}
```

## Grafana Alerting vs Prometheus

| Aspect | Grafana | Prometheus |
|--------|---------|-----------|
| **Creation** | Web UI | YAML files |
| **Evaluation** | In Grafana | In Prometheus |
| **Data sources** | Any (Prometheus, MySQL, etc.) | Prometheus only |
| **Routing** | Limited | Full AlertManager |
| **State persistence** | Yes (in database) | No |
| **Flexibility** | Good for dashboards | Good for infrastructure |

## Common Alert Settings

| Setting | Typical Value | Purpose |
|---------|---------------|---------|
| Evaluation interval | 1m | How often to check |
| For duration | 5m | Wait before firing |
| Repeat interval | 4h | Resend if still firing |
| If no data | Set to NoData | Handle missing data |
| Group by | alertname | How to group alerts |

## Troubleshooting Commands

| Issue | Command |
|-------|---------|
| Check alert evaluation | `curl http://localhost:3000/api/v1/rules` |
| Get alert history | `curl -H "Authorization: Bearer TOKEN" http://localhost:3000/api/v1/rules/{uid}` |
| Check notification | `docker logs grafana \| grep -i notif` |
| Verify condition | Test metric in Prometheus first |
| List all rules | `curl http://localhost:3000/api/ruler/grafana/rules` |

## Best Practices

✅ **DO**:
- Test condition in Prometheus first
- Set appropriate "For" duration
- Add meaningful labels for routing
- Document runbook link
- Include severity level
- Test alert firing

❌ **DON'T**:
- Use overly sensitive thresholds
- Forget to set "For" duration
- Create alerts without testing
- Use ambiguous summaries
- Alert on every metric change
- Overload notification channels

---

**Keep this handy while creating alerts!**
