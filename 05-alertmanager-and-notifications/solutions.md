# Solutions: AlertManager and Notifications

## Solution 1: Access AlertManager UI
**Steps**:
1. Ensure AlertManager running: `docker ps | grep alertmanager`
2. Open browser: `http://localhost:9093`
3. Default port is 9093

**Explanation**: 
- AlertManager UI shows active alerts
- Silences tab for muting alerts
- Status tab shows configuration
- No alerts initially (until Prometheus sends some)

**Expected**: Dashboard with "Alerts", "Silences", "Status" tabs

---

## Solution 2: View AlertManager Configuration
**Command**:
```bash
curl http://localhost:9093/api/v1/status | jq '.config'
```

**Pretty print**:
```bash
curl http://localhost:9093/api/v1/status | jq '.data.config' -r
```

**Explanation**: 
- Returns loaded AlertManager configuration
- Shows all routes, receivers, templates
- Useful for verifying config loaded correctly
- JSON format for programmatic access

**Expected Output**:
```json
{
  "global": {
    "resolve_timeout": "5m"
  },
  "route": {
    "receiver": "default",
    "group_by": ["alertname"]
  },
  "receivers": [
    {
      "name": "default",
      "webhook_configs": [
        {"url": "http://localhost:5001/"}
      ]
    }
  ]
}
```

---

## Solution 3: Create Basic alertmanager.yml
**File content**:
```yaml
global:
  resolve_timeout: 5m

route:
  receiver: 'default'
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h

receivers:
  - name: 'default'
    webhook_configs:
      - url: 'http://localhost:5001/'
```

**Explanation**: 
- `global`: Default settings (timeout before resolving)
- `route`: How to route alerts (default receiver)
- `group_by`: Group alerts by label (alertname)
- `receivers`: Where to send alerts (webhook)

**Validation**:
```bash
docker run --rm -v $(pwd)/alertmanager.yml:/alertmanager.yml \
  prom/alertmanager --config.file=/alertmanager.yml --dry-run
```

---

## Solution 4: Configure Default Route
**In alertmanager.yml**:
```yaml
route:
  receiver: 'default'
  group_by: ['alertname']
  group_wait: 10s        # Wait 10s to batch alerts
  group_interval: 10s    # Send updates every 10s
  repeat_interval: 12h   # Repeat every 12h
```

**Explanation**: 
- `group_by`: Alert grouping dimension
- `group_wait`: Initial wait before sending notification
- `group_interval`: How often to send updates for group
- `repeat_interval`: Resend notification after this duration
- Prevents alert storms while keeping updates

**Effect**:
- Multiple alerts grouped by alertname
- Notifications sent together instead of separately
- Reduces notification noise

---

## Solution 5: Create Webhook Receiver
**webhook.py**:
```python
from http.server import HTTPServer, BaseHTTPRequestHandler
import json
from datetime import datetime

class AlertHandler(BaseHTTPRequestHandler):
    def do_POST(self):
        # Read request body
        length = int(self.headers['Content-Length'])
        body = self.rfile.read(length)
        alerts = json.loads(body)
        
        # Print alerts
        timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
        print(f"\n[{timestamp}] Received {len(alerts.get('alerts', []))} alert(s)")
        
        for alert in alerts.get('alerts', []):
            print(f"  - {alert['labels'].get('alertname')}: {alert['status']}")
        
        # Send OK response
        self.send_response(200)
        self.end_headers()

if __name__ == '__main__':
    print("Webhook server listening on :5001")
    HTTPServer(('0.0.0.0', 5001), AlertHandler).serve_forever()
```

**Run**:
```bash
python webhook.py
```

**Explanation**: 
- Simple HTTP server receiving POST requests
- Parses JSON body containing alerts
- Prints alert info to console
- Responds with 200 OK

**Expected Output** (when alert fires):
```
[2024-01-15 10:30:45] Received 1 alert(s)
  - HighCPUUsage: firing
```

---

## Solution 6: Connect Prometheus to AlertManager
**In prometheus.yml**:
```yaml
global:
  scrape_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

rule_files:
  - /etc/prometheus/rules.yml

scrape_configs:
  # ... scrape configs ...
```

**Command to restart**:
```bash
docker-compose restart prometheus
```

**Verification**:
```bash
# Check logs
docker logs prometheus | grep -i alertmanager

# Check in UI
curl http://localhost:9090/api/v1/alerts | jq '.data.alerts[0]'
```

**Explanation**: 
- `alertmanagers` section tells Prometheus where to send alerts
- Multiple AlertManager instances for HA
- Prometheus sends firing alerts to AlertManager
- AlertManager handles routing and notifications

---

## Solution 7: Trigger Test Alert
**Steps**:
```bash
# 1. Verify alert rule exists in rules.yml
curl http://localhost:9090/api/v1/rules | jq '.data.groups'

# 2. Stop node-exporter to trigger up==0 alert
docker stop node-exporter

# 3. Wait 1-2 minutes for alert to fire (depends on "for:" duration)

# 4. Check Prometheus alerts
curl http://localhost:9090/api/v1/alerts | jq '.data.alerts'

# 5. Check AlertManager
curl http://localhost:9093/api/v1/alerts | jq '.data'

# 6. View in UI
# Browser: http://localhost:9093 (should show alert)
```

**Explanation**: 
- Alert needs time to transition from Inactive → Pending → Firing
- "for" duration must elapse before firing
- AlertManager receives firing alerts only
- Can verify at each step with API

---

## Solution 8: View Alert Routing
**In AlertManager UI**:
1. Go to: `http://localhost:9093`
2. Click "Alerts" tab
3. Expand alert entry
4. See labels, status, receiver, grouping

**Via API**:
```bash
curl http://localhost:9093/api/v1/alerts | jq '.data[] | {
  alertname: .labels.alertname,
  status: .status,
  receiver: .receiver,
  labels: .labels
}'
```

**Explanation**: 
- Shows which receiver alert was sent to
- Shows all labels (used for routing decisions)
- Shows alert status (firing, resolved)
- Receiver determined by route matching

**Expected Output**:
```json
{
  "alertname": "TargetDown",
  "status": "firing",
  "receiver": "default",
  "labels": {
    "severity": "critical",
    "instance": "node-exporter:9100"
  }
}
```

---

## Solution 9: Configure Multiple Routes
**In alertmanager.yml**:
```yaml
route:
  receiver: 'default'
  group_by: ['alertname']
  routes:
    - match:
        severity: critical
      receiver: 'critical'
      continue: false

    - match:
        severity: warning
      receiver: 'warnings'
      continue: false

receivers:
  - name: 'default'
    webhook_configs:
      - url: 'http://localhost:5001/'

  - name: 'critical'
    webhook_configs:
      - url: 'http://critical-webhook:5001/'

  - name: 'warnings'
    webhook_configs:
      - url: 'http://warning-webhook:5001/'
```

**Explanation**: 
- Hierarchical routing tree
- Alert matches first route where condition true
- `continue: false` prevents further route matching
- Different receivers for different severities

---

## Solution 10: Test Alert Inhibition
**In alertmanager.yml**:
```yaml
inhibit_rules:
  - source_match:
      severity: critical
    target_match:
      severity: warning
    equal: ['instance']

receivers:
  - name: 'default'
```

**Explanation**: 
- When source (critical alert) fires for instance X
- Suppress target (warning alert) for same instance X
- Prevents alert noise when critical incident ongoing
- `equal` specifies which labels must match

**Example**:
```
If Critical alert for instance="prod-01" fires
  → Suppress Warning alert for instance="prod-01"
  → But Warning for instance="prod-02" still shows
```

**Testing**:
```bash
# Fire both critical and warning alerts
# Check UI: warning should be suppressed/hidden
curl http://localhost:9093/api/v1/alerts
```

---

**All Solutions Complete ✅**
