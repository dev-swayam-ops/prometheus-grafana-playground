# Solutions: Grafana Basics

## Solution 1: Start Grafana in Docker
**Commands**:
```bash
docker run -d --name grafana -p 3000:3000 grafana/grafana:latest
docker ps | grep grafana
curl http://localhost:3000
```

**Explanation**: 
- `-d` runs in background (daemon mode)
- `--name grafana` assigns container name
- `-p 3000:3000` maps port 3000
- `grafana/grafana:latest` is official Grafana image

**Expected Output**:
```
Container ID: abc123def456...
grafana  grafana/grafana  "grafana server" 3000/tcp  grafana

HTTP/1.1 200 OK
(Returns HTML login page)
```

---

## Solution 2: Login to Grafana
**Steps**:
1. Browser: `http://localhost:3000`
2. Username: `admin`
3. Password: `admin`
4. Click "Log in"

**Explanation**: 
- Default credentials configured in image
- First login recommended to change password
- Home dashboard shows after successful login
- Top right shows profile icon after login

**Expected**: Welcome dashboard, "Home" label visible

---

## Solution 3: Change Admin Password
**Steps**:
1. Click profile icon (top right corner)
2. Select "Change password"
3. Old password: `admin`
4. New password: `yourpassword`
5. Click "Update password"

**Explanation**: 
- Security best practice to change defaults
- Password must meet minimum requirements
- Change takes effect immediately
- Must re-login with new password

**Expected Output**: "Password updated" message

---

## Solution 4: Access Grafana API
**Commands**:
```bash
# Check health
curl http://localhost:3000/api/health

# Get organization info
curl http://localhost:3000/api/org
```

**Explanation**: 
- `/api/health` returns server health status
- `/api/org` returns current organization
- API requires authentication for sensitive endpoints
- Useful for scripting and automation

**Expected Output**:
```json
{
  "status": "ok",
  "database": "ok"
}

{
  "id": 1,
  "name": "Main Org.",
  "address": {...}
}
```

---

## Solution 5: Add Prometheus Data Source
**Steps**:
1. Click "Connections" (left sidebar)
2. Click "Data sources"
3. Click "Add data source"
4. Select "Prometheus"
5. URL: `http://localhost:9090`
6. Click "Save & test"

**Explanation**: 
- Data sources connect Grafana to metrics
- Multiple data sources supported (Prometheus, MySQL, etc.)
- URL must be reachable from Grafana container
- "Save & test" verifies connectivity

**Expected**: Green "Data source is working" message

---

## Solution 6: List Data Sources via API
**Command**:
```bash
curl http://localhost:3000/api/datasources
```

**Pretty print**:
```bash
curl http://localhost:3000/api/datasources | jq '.'
```

**Explanation**: 
- Returns array of all data sources
- Shows data source type, name, URL
- Useful for provisioning/automation
- Can filter by type or search

**Expected Output**:
```json
[
  {
    "id": 1,
    "uid": "prometheus",
    "name": "Prometheus",
    "type": "prometheus",
    "url": "http://localhost:9090"
  }
]
```

---

## Solution 7: Create Simple Dashboard
**Steps**:
1. Click "+" (left sidebar) → "Dashboard"
2. Click "Add panel"
3. Data source: select "Prometheus"
4. Query field: `up`
5. Click "Apply"
6. Click "Save" (top right)
7. Title: "Test Dashboard"

**Explanation**: 
- Dashboard is collection of panels
- Each panel displays one metric/query
- Apply saves panel to dashboard
- Save persists dashboard to Grafana database

**Expected**: Graph showing target up/down status

---

## Solution 8: Explore Panel Editor
**Steps**:
1. Open dashboard with panel
2. Click gear icon (top right of panel)
3. Explore sections:
   - **Query**: Change metric/data source
   - **Visualization**: Choose chart type
   - **Field**: Format numbers/units
   - **Options**: Panel-specific settings
4. Try changing visualization to "Stat"

**Explanation**: 
- Panel editor customizes individual panels
- Visualization changes how data displays
- Field section handles formatting
- Changes persist after "Apply"

---

## Solution 9: Search Dashboards
**Method 1 - UI Search**:
1. Click search icon (⌘K or Ctrl+K)
2. Type dashboard name
3. See results and click to open

**Method 2 - API Search**:
```bash
curl http://localhost:3000/api/search?query=dashboard_name
```

**Explanation**: 
- Search finds dashboards, panels, folders
- Keyboard shortcut (⌘K) for quick access
- Useful when many dashboards exist

**Expected**: List of matching dashboards

---

## Solution 10: View Grafana Configuration
**Commands**:
```bash
# View logs
docker logs grafana | tail -30

# Check container details
docker inspect grafana | grep -E "Status|Health"

# Check running processes
docker exec grafana ps aux

# View Grafana version
curl http://localhost:3000/api/health | jq '.version'
```

**Explanation**: 
- Logs show startup messages, errors, warnings
- Health status indicates if services running
- Useful for debugging connectivity issues
- Version info for compatibility checking

**Expected Output** (Logs):
```
level=info msg="Starting Grafana" version=X.X.X
level=info msg="Database" type=sqlite3
level=info msg="HTTP Server listen" addr=0.0.0.0:3000
```

---

**All Solutions Complete ✅**
