# Solutions: Grafana Alerting

## Solution 1: Access Grafana Alert Rules
**Steps**:
1. Open browser: `http://localhost:3000`
2. Click "Alerting" menu (left sidebar)
3. Click "Alert rules"

**Explanation**: 
- Alert rules page shows all configured Grafana alerts
- Shows rule name, status, last evaluation
- Status: Normal, Pending, Firing, Error
- Green = Normal, Yellow = Pending, Red = Firing

**Expected**: Dashboard showing list of alert rules with status indicators

---

## Solution 2: Create Simple Alert Rule
**Steps**:
1. Open dashboard
2. Click on any panel
3. Click gear icon (panel settings)
4. Click "Alert" tab
5. Click "Create alert rule"
6. Fill in rule details
7. Click "Save"

**Explanation**: 
- Alert rules created at panel level in Grafana
- Can be created from any panel with data
- Rule evaluated on schedule (e.g., every 1 minute)
- Initial status is Normal

**Expected**: Alert rule created, shown in Alert rules page

---

## Solution 3: Set Alert Labels
**In alert editor**:
```yaml
Labels:
  severity: critical
  team: devops
  env: prod
  service: api
```

**Explanation**: 
- Labels add metadata to alerts
- Used for routing in notification policies
- Can add any custom labels
- Labels shown in alert details and notifications

**Expected**: Labels save with alert rule

---

## Solution 4: Add Alert Annotations
**In alert editor**:
```yaml
Annotations:
  Summary: "CPU usage high: {{ $value }}%"
  Description: "CPU on {{ $labels.instance }} exceeded threshold"
  Runbook: "https://wiki.example.com/high-cpu"
  Grafana: "{{ grafana_dashboard_url }}"
```

**Template Variables**:
- `{{ $value }}` = metric value
- `{{ $labels.instance }}` = label value
- `{{ grafana_dashboard_url }}` = dashboard link

**Explanation**: 
- Annotations provide human-readable text
- Template variables interpolate at runtime
- Used in notifications
- Runbook link directs to documentation

---

## Solution 5: Set Alert For Duration
**Configuration**:
```
Evaluation interval: 1m
For: 5m (must be true for 5 minutes)
If no data: Set state to "NoData"
```

**State Transitions**:
```
Normal → (condition true) → Pending (1-5 min)
       → (after 5 min) → Firing (alert active)
       → (condition false) → Resolved
```

**Explanation**: 
- "For" prevents false positives
- Condition must be true continuously
- "If no data" handles missing data
- Prevents alerts from unstable metrics

---

## Solution 6: Configure Notification Policy
**Steps**:
1. In alert rule editor
2. Scroll to "Notification policy"
3. Select receiver: "AlertManager"
4. Set repeat interval: 4h
5. Save rule

**Explanation**: 
- Notification policy routes alert to receiver
- Repeat interval resends if still firing
- Can integrate with AlertManager
- Different policies for different severities

---

## Solution 7: Query Alert Status via API
**Command**:
```bash
# Get API token first
TOKEN=$(curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"user":"admin","password":"admin"}' | jq -r '.token')

# Get alert rules
curl -H "Authorization: Bearer $TOKEN" \
  http://localhost:3000/api/ruler/grafana/rules
```

**Explanation**: 
- Returns all alert rules in JSON format
- Shows rule name, condition, labels
- API requires authentication token
- Useful for automation/scripting

---

## Solution 8: Test Alert Firing
**Steps**:
1. Open alert rule in Grafana
2. Click "Edit"
3. In condition, lower threshold (e.g., from 80 to current value)
4. Save → Alert should fire
5. Go to "Alert rules" → Check status: "Firing"
6. Edit again, reset threshold to original

**Explanation**: 
- Testing verifies alert condition works
- Lowering threshold ensures condition is met
- Status changes to "Firing" when true
- For duration must elapse (unless set to 0)

**Expected**: Alert status changes to Firing

---

## Solution 9: View Alert History
**Steps**:
1. Open alert rule details
2. Click "Instances" tab
3. See list of alert firing events
4. Click on specific instance for details
5. View state transitions and times

**Explanation**: 
- Instances show each alert activation
- Shows when alert fired and resolved
- Displays value at time of firing
- Useful for incident investigation

**Expected**: List of alert instances with timestamps

---

## Solution 10: Disable Alert Rule
**Steps**:
1. Open alert rule
2. Click "Pause" button (or three dots menu)
3. Confirm: Alert paused message
4. Status shows "Paused" 
5. Click button again to re-enable

**Explanation**: 
- Pausing disables evaluation
- Alert won't fire while paused
- Useful during maintenance
- Can pause/resume without deleting

**Expected**: Alert rule status shows "Paused", rule not evaluated

---

**All Solutions Complete ✅**
