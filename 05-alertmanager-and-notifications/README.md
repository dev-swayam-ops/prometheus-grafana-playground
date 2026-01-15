# 05 - AlertManager and Notifications

## What You'll Learn
- AlertManager architecture and components
- Setting up AlertManager with Prometheus
- Configuring notification routes
- Sending alerts to Slack, Email, PagerDuty
- Alert grouping, inhibition, and silencing
- Testing alerts end-to-end
- Webhook integration basics

## Prerequisites
- [04-recording-rules-and-alerting](../04-recording-rules-and-alerting/README.md) completed
- Alert rules configured in Prometheus
- Slack workspace (optional, for testing)
- Email address or webhook service (ngrok/webhook.site)
- Docker and Docker Compose installed

## Key Concepts

### AlertManager Flow
```
Prometheus Alert (firing) 
    ↓
AlertManager receives
    ↓
Match against routes
    ↓
Apply inhibition/silencing
    ↓
Group alerts
    ↓
Send notifications (Slack, Email, etc.)
```

### Configuration Hierarchy

**1. Global**: Default settings
```yaml
global:
  resolve_timeout: 5m  # How long until alert auto-resolves
  slack_api_url: 'https://hooks.slack.com/...'
```

**2. Route**: Tree of routing rules
```yaml
route:
  receiver: 'default'      # Default receiver
  group_by: ['alertname']  # Group by alert name
  group_wait: 10s          # Wait before sending
  group_interval: 10s      # Send updates every 10s
  repeat_interval: 12h     # Repeat every 12h
  routes:                  # Child routes
    - match:
        severity: critical
      receiver: 'critical'
```

**3. Receivers**: Notification channels
```yaml
receivers:
  - name: 'default'
    slack_configs:
      - channel: '#alerts'
  
  - name: 'critical'
    email_configs:
      - to: 'admin@example.com'
```

## Hands-on Lab: AlertManager Setup

### Step 1: Create alertmanager.yml
```yaml
global:
  resolve_timeout: 5m

route:
  receiver: 'console'
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h

receivers:
  - name: 'console'
    webhook_configs:
      - url: 'http://localhost:5001/'
```

### Step 2: Create test webhook receiver (webhook.py)
```python
from http.server import HTTPServer, BaseHTTPRequestHandler
import json

class AlertHandler(BaseHTTPRequestHandler):
    def do_POST(self):
        content_length = int(self.headers['Content-Length'])
        body = self.rfile.read(content_length)
        alerts = json.loads(body)
        
        print(f"\n[ALERT] Received {len(alerts.get('alerts', []))} alert(s)")
        for alert in alerts.get('alerts', []):
            print(f"  - {alert['labels'].get('alertname', 'Unknown')}: {alert['status']}")
        
        self.send_response(200)
        self.end_headers()

if __name__ == '__main__':
    server = HTTPServer(('0.0.0.0', 5001), AlertHandler)
    print("Webhook receiver listening on :5001")
    server.serve_forever()
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
    depends_on:
      - alertmanager

  alertmanager:
    image: prom/alertmanager:latest
    container_name: alertmanager
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
    command:
      - --config.file=/etc/alertmanager/alertmanager.yml

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
prometheus        prom/prometheus:latest     Up
alertmanager      prom/alertmanager:latest   Up
node-exporter     prom/node-exporter:latest  Up
```

### Step 5: Access AlertManager UI
Open: `http://localhost:9093`

**Expected**: AlertManager dashboard with 0 active alerts

### Step 6: Trigger a Test Alert
Manually trigger alert in Prometheus:
1. Go to `http://localhost:9090/graph`
2. Enter: `up{job="node"} == 0` 
3. Force alert by stopping node-exporter: `docker stop node-exporter`

**Expected**: Alert appears in AlertManager UI

### Step 7: Check AlertManager API
```bash
curl http://localhost:9093/api/v1/alerts | jq '.data'
```

**Expected Output**: JSON with active alerts

### Step 8: View Grouped Alerts
In AlertManager UI, see how alerts are grouped by alert name and severity

## Validation

Verify setup:
```bash
# Check AlertManager config
curl http://localhost:9093/api/v1/status

# View alerts in AlertManager
curl http://localhost:9093/api/v1/alerts

# Check Prometheus knows about AlertManager
curl http://localhost:9090/api/v1/alerts | jq '.data.alerts'
```

## Cleanup

```bash
docker-compose down
docker-compose rm -v
```

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| Alerts not reaching AlertManager | Check prometheus.yml has alerting config |
| AlertManager config not valid | Validate YAML: `yamllint alertmanager.yml` |
| Webhook not receiving alerts | Ensure AlertManager can reach webhook URL |
| Duplicate notifications | Reduce `repeat_interval` or increase `group_interval` |
| Alerts won't group | Add to `group_by` in route config |

## Troubleshooting

**Issue**: AlertManager not starting
```bash
docker logs alertmanager

# Check config
docker run --rm -v $(pwd)/alertmanager.yml:/alertmanager.yml \
  prom/alertmanager --config.file=/alertmanager.yml --dry-run
```

**Issue**: Alerts not triggering
```bash
# Check Prometheus can reach AlertManager
docker exec prometheus curl http://alertmanager:9093

# View alert evaluation
curl http://localhost:9090/api/v1/query?query=ALERTS
```

**Issue**: Webhook not receiving data
```bash
# Check AlertManager logs for errors
docker logs alertmanager | grep -i webhook

# Verify route config
curl http://localhost:9093/api/v1/status | jq '.config'
```

## Next Steps

✅ AlertManager and notifications configured!

**Ready to proceed to**:
- [06-grafana-basics](../06-grafana-basics/README.md) - Visualize alerts in Grafana
- [exercises](./exercises.md) - Practice with hands-on exercises
