# Exercises: AlertManager and Notifications

## Exercise 1: Access AlertManager UI (Easy)
Open AlertManager web interface and explore.

**Task**:
1. Start AlertManager with Docker
2. Open: `http://localhost:9093`
3. Explore: Alerts, Silences, Status tabs

**Validation**: AlertManager UI loads with 0 alerts (initially).

---

## Exercise 2: View AlertManager Configuration (Easy)
Check loaded AlertManager configuration.

**Task**:
```bash
curl http://localhost:9093/api/v1/status | jq '.config'
```

**Validation**: Returns JSON with receivers, routes, and global config.

---

## Exercise 3: Create Basic alertmanager.yml (Easy)
Set up minimal AlertManager config.

**Task**:
```yaml
global:
  resolve_timeout: 5m

route:
  receiver: 'default'

receivers:
  - name: 'default'
    webhook_configs:
      - url: 'http://localhost:5001/'
```

**Validation**: AlertManager starts without config errors.

---

## Exercise 4: Configure Default Route (Easy)
Set up route that groups all alerts.

**Task**:
```yaml
route:
  receiver: 'default'
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
```

**Validation**: AlertManager loads config successfully.

---

## Exercise 5: Create Webhook Receiver (Medium)
Set up webhook endpoint to receive alerts.

**Task**:
Create Python script to listen for webhooks:
```python
from http.server import HTTPServer, BaseHTTPRequestHandler
import json

class Handler(BaseHTTPRequestHandler):
    def do_POST(self):
        length = int(self.headers['Content-Length'])
        body = json.loads(self.rfile.read(length))
        print(f"Received {len(body.get('alerts', []))} alerts")
        self.send_response(200)
        self.end_headers()

HTTPServer(('0.0.0.0', 5001), Handler).serve_forever()
```

Run: `python webhook.py`

**Validation**: Script runs on port 5001.

---

## Exercise 6: Connect Prometheus to AlertManager (Medium)
Configure Prometheus to send alerts to AlertManager.

**Task**:
In `prometheus.yml`:
```yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']
```

Restart Prometheus.

**Validation**: Prometheus UI shows AlertManager status.

---

## Exercise 7: Trigger Test Alert (Medium)
Force a Prometheus alert to AlertManager.

**Task**:
1. Add alert rule with low threshold
2. Stop node-exporter: `docker stop node-exporter`
3. Wait for alert to fire
4. Check AlertManager UI for alert

**Validation**: Alert appears in AlertManager Alerts tab.

---

## Exercise 8: View Alert Routing (Medium)
Check how AlertManager routes alerts.

**Task**:
In AlertManager UI:
1. Click "Alerts" tab
2. Expand alert to see labels and receivers
3. Check which receiver it was sent to

**Validation**: Alert shows routing details (receiver name, grouping).

---

## Exercise 9: Configure Multiple Routes (Medium)
Set up different routes for different severities.

**Task**:
```yaml
route:
  receiver: 'default'
  routes:
    - match:
        severity: critical
      receiver: 'critical'
    - match:
        severity: warning
      receiver: 'warnings'
```

**Validation**: Alerts route to correct receivers based on severity.

---

## Exercise 10: Test Alert Inhibition (Medium)
Set up inhibition rule to suppress alerts.

**Task**:
```yaml
inhibit_rules:
  - source_match:
      severity: critical
    target_match:
      severity: warning
    equal: ['instance']
```

This prevents warning when critical exists for same instance.

**Validation**: Warning alert suppressed when critical alert fires.

---

**All Exercises Complete ✅**
