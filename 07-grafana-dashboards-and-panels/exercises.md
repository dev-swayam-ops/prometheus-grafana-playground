# Exercises: Grafana Dashboards and Panels

## Exercise 1: Create New Dashboard (Easy)
Create a blank dashboard and save it.

**Task**:
1. Click "+" → "Dashboard"
2. Click "Add panel"
3. Select Prometheus data source
4. Enter metric: `up`
5. Click "Apply"
6. Click "Save" → "Dashboard Name"

**Validation**: Dashboard created and saved successfully.

---

## Exercise 2: Add Graph Panel (Easy)
Add a time-series graph showing CPU usage.

**Task**:
Title: "CPU Usage"
Metric: `100 * (1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m])))`
Visualization: Graph
Unit: Percent

**Validation**: Graph displays CPU percentage over time.

---

## Exercise 3: Add Gauge Panel (Easy)
Add a gauge showing memory percentage.

**Task**:
Title: "Memory Usage"
Metric: `(1 - node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes) * 100`
Visualization: Gauge
Unit: Percent
Max: 100

**Validation**: Gauge shows 0-100% with color coding.

---

## Exercise 4: Add Stat Panel (Easy)
Add a stat panel showing current uptime.

**Task**:
Title: "System Uptime"
Metric: `node_boot_time_seconds`
Visualization: Stat
Value calculation: Last
Unit: None

**Validation**: Panel shows latest boot time value.

---

## Exercise 5: Change Visualization Type (Medium)
Modify panel visualization from graph to bar chart.

**Task**:
1. Open any existing panel
2. Click gear icon (panel settings)
3. Click "Visualization" tab
4. Select "Bar chart"
5. Click "Apply"

**Validation**: Panel visualization changes to bar chart.

---

## Exercise 6: Add Dashboard Variable (Medium)
Create a variable for filtering by instance.

**Task**:
1. Click "Dashboard settings" (gear icon)
2. Click "Variables" tab
3. Click "Add variable"
4. Name: `instance`
5. Type: Query
6. Query: `label_values(up, instance)`
7. Click "Add"

**Validation**: Variable dropdown appears at top of dashboard.

---

## Exercise 7: Use Variable in Panel Query (Medium)
Update panel query to use the instance variable.

**Task**:
1. Open panel editor
2. Change query to: `up{instance="$instance"}`
3. Click "Apply"
4. Test dropdown filter

**Validation**: Panel updates when variable selection changes.

---

## Exercise 8: Add Table Panel (Medium)
Create a table showing all metrics for selected instance.

**Task**:
Title: "Target Status"
Query: `label_values(up, instance)`
Visualization: Table
Columns: Instant metric values

**Validation**: Table displays all matching metrics.

---

## Exercise 9: Configure Panel Legend (Medium)
Customize legend display in graph panel.

**Task**:
1. Open graph panel settings
2. Click "Options" tab
3. Find "Legend" section
4. Enable/disable: Show legend, Show values
5. Change legend placement: bottom/right

**Validation**: Legend display changes in panel.

---

## Exercise 10: Export Dashboard as JSON (Medium)
Export dashboard configuration as JSON file.

**Task**:
1. Open dashboard
2. Click "Dashboard settings" (gear)
3. Click "Share dashboard"
4. Click "Export" tab
5. Click "Save to file"

**Validation**: JSON file downloads with dashboard config.

---

**All Exercises Complete ✅**
