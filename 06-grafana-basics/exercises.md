# Exercises: Grafana Basics

## Exercise 1: Start Grafana in Docker (Easy)
Run Grafana container and verify it's accessible.

**Task**:
```bash
docker run -d --name grafana -p 3000:3000 grafana/grafana:latest
docker ps | grep grafana
curl http://localhost:3000
```

**Validation**: Container running, HTTP 200 response from Grafana.

---

## Exercise 2: Login to Grafana (Easy)
Access Grafana UI and login with default credentials.

**Task**:
1. Open browser: `http://localhost:3000`
2. Login: admin / admin
3. See home dashboard

**Validation**: Home dashboard displays, no error messages.

---

## Exercise 3: Change Admin Password (Easy)
Set a new admin password for security.

**Task**:
1. Click profile icon (top right)
2. Select "Change password"
3. Enter old password: admin
4. Enter new password
5. Confirm password
6. Click "Update password"

**Validation**: Success message appears, new password works.

---

## Exercise 4: Access Grafana API (Easy)
Query Grafana REST API to verify it's working.

**Task**:
```bash
curl http://localhost:3000/api/health
curl http://localhost:3000/api/org
```

**Validation**: Returns JSON with health/org info.

---

## Exercise 5: Add Prometheus Data Source (Medium)
Configure Prometheus as a data source in Grafana.

**Task**:
1. Click "Connections" → "Data sources"
2. Click "Add data source"
3. Select "Prometheus"
4. Set URL: `http://localhost:9090`
5. Click "Save & test"

**Validation**: "Data source is working" message appears.

---

## Exercise 6: List Data Sources via API (Medium)
Query Grafana API to see configured data sources.

**Task**:
```bash
curl http://localhost:3000/api/datasources
```

**Validation**: Returns JSON array with Prometheus data source.

---

## Exercise 7: Create Simple Dashboard (Medium)
Create a new dashboard with a basic panel.

**Task**:
1. Click "+" → "Dashboard"
2. Click "Add panel"
3. Select Prometheus data source
4. Enter metric: `up`
5. Click "Apply"
6. Save dashboard: "Test Dashboard"

**Validation**: Dashboard created, panel shows `up` metric graph.

---

## Exercise 8: Explore Panel Editor (Medium)
Investigate panel configuration options.

**Task**:
1. Open any panel
2. Click gear icon (panel settings)
3. Explore tabs: Query, Visualization, Field, Options
4. Change visualization to "Stat"
5. Click "Apply"

**Validation**: Panel visualization changes, new format displayed.

---

## Exercise 9: Search Dashboards (Medium)
Use Grafana search to find dashboards.

**Task**:
1. Click search icon (left sidebar)
2. Search: "dashboard_name" or press Ctrl+K
3. See search results

**Validation**: Search returns matching dashboards.

---

## Exercise 10: View Grafana Configuration (Medium)
Check Grafana system information and logs.

**Task**:
```bash
# Check Grafana logs
docker logs grafana | tail -20

# Check container health
docker inspect grafana | grep -i health

# Check running processes
docker exec grafana ps aux
```

**Validation**: Logs show Grafana startup messages, health status OK.

---

**All Exercises Complete ✅**
