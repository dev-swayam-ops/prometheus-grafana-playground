# Cheatsheet: AlertManager and Notifications

## AlertManager Configuration Structure

```yaml
global:
  resolve_timeout: 5m
  slack_api_url: 'https://hooks.slack.com/...'
  pagerduty_url: 'https://events.pagerduty.com/v2/enqueue'

route:
  receiver: 'default'
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  routes:
    - match:
        severity: critical
      receiver: 'critical'

receivers:
  - name: 'default'
  - name: 'critical'

inhibit_rules:
  - source_match:
      severity: critical
    target_match:
      severity: warning
    equal: ['instance']
```

## Route Configuration

| Field | Purpose | Example |
|-------|---------|---------|
| `receiver` | Default receiver | `'default'`, `'slack'` |
| `group_by` | Grouping dimension | `['alertname', 'instance']` |
| `group_wait` | Initial batch wait | `10s`, `30s` |
| `group_interval` | Update send interval | `5m` |
| `repeat_interval` | Resend notification | `4h`, `12h` |
| `match` | Route condition | `{severity: critical}` |
| `match_re` | Regex condition | `{job: "^prom.*"}` |
| `continue` | Try next route | `true` or `false` |
| `routes` | Child routes | Hierarchical tree |

## Receiver Types

| Type | Config | Purpose |
|------|--------|---------|
| **Slack** | `slack_configs` | Slack workspace |
| **Email** | `email_configs` | Email address |
| **PagerDuty** | `pagerduty_configs` | On-call management |
| **OpsGenie** | `opsgenie_configs` | Alert management |
| **Webhook** | `webhook_configs` | Custom HTTP endpoint |

## Slack Receiver Example

```yaml
receivers:
  - name: 'slack'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'
        channel: '#alerts'
        title: '{{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
        send_resolved: true
```

## Email Receiver Example

```yaml
receivers:
  - name: 'email'
    email_configs:
      - to: 'alerts@example.com'
        from: 'alertmanager@example.com'
        smarthost: 'smtp.gmail.com:587'
        auth_username: 'your-email@gmail.com'
        auth_password: 'your-password'
        headers:
          Subject: '{{ .GroupLabels.alertname }}'
```

## Webhook Receiver Example

```yaml
receivers:
  - name: 'webhook'
    webhook_configs:
      - url: 'http://webhook-receiver:5001/alerts'
        send_resolved: true
```

## Inhibition Rules

| Field | Purpose | Example |
|-------|---------|---------|
| `source_match` | When this alert fires | `{severity: critical}` |
| `target_match` | Suppress this alert | `{severity: warning}` |
| `equal` | Match on these labels | `['instance', 'job']` |

## AlertManager API Endpoints

| Endpoint | Purpose | Method |
|----------|---------|--------|
| `/api/v1/status` | Get configuration | GET |
| `/api/v1/alerts` | List all alerts | GET |
| `/api/v1/alerts/groups` | Grouped alerts | GET |
| `/api/v1/silences` | List silences | GET |
| `/api/v1/silence` | Create silence | POST |

## curl Commands

**Get status**:
```bash
curl http://localhost:9093/api/v1/status | jq '.data'
```

**List alerts**:
```bash
curl http://localhost:9093/api/v1/alerts | jq '.data'
```

**Get grouped alerts**:
```bash
curl http://localhost:9093/api/v1/alerts/groups | jq '.data'
```

**Create silence** (1 hour):
```bash
curl -X POST http://localhost:9093/api/v1/silences \
  -d '{
    "matchers": [{"name": "alertname", "value": "HighCPU"}],
    "duration": "1h"
  }' \
  -H 'Content-Type: application/json'
```

## Alert Grouping Example

**Without grouping** (send 5 separate notifications):
```
Alert 1: HighCPU on host1
Alert 2: HighCPU on host2
Alert 3: HighMemory on host1
Alert 4: HighMemory on host2
Alert 5: HighDisk on host1
```

**With group_by: ['alertname']** (send 3 grouped notifications):
```
HighCPU (2 alerts):
  - host1
  - host2

HighMemory (2 alerts):
  - host1
  - host2

HighDisk (1 alert):
  - host1
```

## Alert States

| State | Meaning | Duration |
|-------|---------|----------|
| **firing** | Alert condition true | Active until resolved |
| **resolved** | Alert condition false | Transient |
| **pending** | (Prometheus state) | Until "for:" duration |

## Template Variables

| Variable | Example | Use |
|----------|---------|-----|
| `{{ .GroupLabels.alertname }}` | `HighCPU` | Alert name |
| `{{ .Labels.instance }}` | `host1:9100` | Alert label |
| `{{ .Annotations.description }}` | "CPU at 95%" | Description |
| `{{ .Status }}` | `firing` or `resolved` | Alert status |
| `{{ .Alerts \| len }}` | `3` | Alert count in group |

## Silencing Alerts

**Via UI**:
1. Go to: `http://localhost:9093`
2. Click "Silences" tab
3. Create New Silence
4. Select alert matchers
5. Set duration

**Via API**:
```bash
curl -X POST http://localhost:9093/api/v1/silences \
  -H 'Content-Type: application/json' \
  -d '{
    "matchers": [
      {"name": "alertname", "value": "HighCPU", "isRegex": false}
    ],
    "startsAt": "2024-01-15T10:00:00Z",
    "endsAt": "2024-01-15T11:00:00Z",
    "createdBy": "admin",
    "comment": "Maintenance window"
  }'
```

## Docker Compose Setup

```yaml
version: '3'
services:
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./rules.yml:/etc/prometheus/rules.yml
    ports:
      - "9090:9090"

  alertmanager:
    image: prom/alertmanager:latest
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
    ports:
      - "9093:9093"
    command:
      - --config.file=/etc/alertmanager/alertmanager.yml
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Alerts not reaching AlertManager | Check prometheus.yml `alerting` section |
| No notifications sent | Check receiver config (webhook URL, credentials) |
| Duplicate notifications | Increase `group_interval` or `repeat_interval` |
| Alert stuck in "firing" | May need manual silence or resolution |
| YAML parse error | Use yamllint: `yamllint alertmanager.yml` |

## Debugging Commands

| Task | Command |
|------|---------|
| Check config | `curl http://localhost:9093/api/v1/status` |
| View all alerts | `curl http://localhost:9093/api/v1/alerts` |
| Check logs | `docker logs alertmanager` |
| Validate YAML | `yamllint alertmanager.yml` |
| Test webhook | `curl -X POST http://webhook:5001/` |

## Quick Setup Template

```yaml
# alertmanager.yml
global:
  resolve_timeout: 5m

route:
  receiver: 'default'
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 4h

receivers:
  - name: 'default'
    webhook_configs:
      - url: 'http://webhook-receiver:5001/'
        send_resolved: true

inhibit_rules: []
```

---

**Keep this handy while configuring AlertManager!**
