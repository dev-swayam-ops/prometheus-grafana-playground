# Exercises: Grafana Alerting

## Exercise 1: Access Grafana Alert Rules (Easy)
View all configured alert rules in Grafana.

**Task**:
1. Open Grafana: `http://localhost:3000`
2. Click "Alerting" → "Alert rules" (left sidebar)
3. See list of alert rules and their status

**Validation**: Alert rules page displays with rules listed.

---

## Exercise 2: Create Simple Alert Rule (Easy)
Create alert rule using UI.

**Task**:
1. Open Grafana dashboard
2. Click on any panel → gear icon → "Edit"
3. Click "Alert" tab
4. Click "Create alert rule"
5. Set threshold and save

**Validation**: Alert rule created, status shows Normal/Pending.

---

## Exercise 3: Set Alert Labels (Easy)
Add custom labels to alert for routing.

**Task**:
In alert rule editor:
```yaml
Labels:
  severity: critical
  team: devops
  env: prod
```

**Validation**: Labels save without errors.

---

## Exercise 4: Add Alert Annotations (Easy)
Include description and summary.

**Task**:
```yaml
Annotations:
  Summary: "Alert summary {{ $value }}"
  Description: "Detailed description"
  Runbook: "https://wiki.example.com/runbook"
```

**Validation**: Annotations display in alert details.

---

## Exercise 5: Set Alert For Duration (Medium)
Configure alert "for" duration before firing.

**Task**:
In condition settings:
- "For": 5 minutes
- "If no data": Set state to "NoData"

Save and observe state transitions.

**Validation**: Alert respects "for" duration, transitions correctly.

---

## Exercise 6: Configure Notification Policy (Medium)
Set up notification routing in alert rule.

**Task**:
1. Click "Notification policy"
2. Select receiver: "AlertManager" or "default"
3. Set repeat interval: 4h
4. Save

**Validation**: Policy saves, shows in alert details.

---

## Exercise 7: Query Alert Status via API (Medium)
Check alert rules using Grafana API.

**Task**:
```bash
curl -H "Authorization: Bearer TOKEN" \
  http://localhost:3000/api/ruler/grafana/rules
```

**Validation**: Returns JSON with alert rules.

---

## Exercise 8: Test Alert Firing (Medium)
Force alert to fire by lowering threshold.

**Task**:
1. Edit alert rule
2. Lower threshold to trigger alert
3. Save
4. Check status: should be "Firing"
5. Reset threshold

**Validation**: Alert state transitions from Normal → Firing.

---

## Exercise 9: View Alert History (Medium)
Check past alert firing events.

**Task**:
1. Open alert rule
2. Click "Instances" tab
3. See list of firing/resolved alerts
4. Click alert for details

**Validation**: Alert history shows state transitions and timestamps.

---

## Exercise 10: Disable Alert Rule (Medium)
Temporarily disable an alert rule.

**Task**:
1. Open alert rule
2. Click "Pause" button
3. Confirm: Rule is disabled
4. Re-enable by clicking button again

**Validation**: Alert rule status shows Paused, resumes after re-enable.

---

**All Exercises Complete ✅**
