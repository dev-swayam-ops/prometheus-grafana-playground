# Solutions: Grafana Dashboards and Panels

## Solution 1: Create New Dashboard
**Steps**:
1. Click "+" (left sidebar) → "Dashboard"
2. Click "Add panel"
3. Data source: "Prometheus"
4. Query: `up`
5. Click "Apply"
6. Click "Save"
7. Title: "My Dashboard"
8. Click "Save"

**Explanation**: 
- "+" button opens quick menu
- "Add panel" creates visualization
- "Apply" saves panel to dashboard
- "Save" persists dashboard to database

**Expected**: Dashboard created with one panel showing target status

---

## Solution 2: Add Graph Panel
**Panel Configuration**:
```
Title: CPU Usage
Data source: Prometheus
Query: 100 * (1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m])))
Visualization: Graph (Time series)
Unit: Percent
Min: 0, Max: 100
```

**Explanation**: 
- Query calculates: (1 - idle_time) * 100 = usage%
- Graph visualization shows trend over time
- Unit formatting displays % symbol
- Min/Max sets axis range

**Expected**: Line graph showing CPU % between 0-100

---

## Solution 3: Add Gauge Panel
**Panel Configuration**:
```
Title: Memory Usage
Data source: Prometheus
Query: (1 - node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes) * 100
Visualization: Gauge
Unit: Percent
Min: 0, Max: 100
Color: Green <50%, Yellow 50-80%, Red >80%
```

**Explanation**: 
- Gauge shows single value on 0-100% scale
- Color coding indicates severity
- Useful for KPIs and utilization metrics
- Shows threshold ranges

**Expected**: Circular gauge with color-coded zones

---

## Solution 4: Add Stat Panel
**Panel Configuration**:
```
Title: System Uptime
Data source: Prometheus
Query: node_boot_time_seconds
Visualization: Stat
Value calculation: Last
Unit: short (or custom)
```

**Explanation**: 
- Stat panel displays single large number
- Last value shows most recent measurement
- Good for KPIs, counts, current metrics
- Can show threshold status

**Expected**: Large number showing current value

---

## Solution 5: Change Visualization Type
**Steps**:
1. Open dashboard with panel
2. Click gear icon (top right of panel)
3. Click "Visualization" tab
4. Select "Bar chart" from list
5. Click "Apply"

**Explanation**: 
- All panels support multiple visualizations
- Visualization tab shows available types
- Changes format without losing data
- Some types need specific data format

**Expected**: Panel changes from graph to bar chart

---

## Solution 6: Add Dashboard Variable
**Steps**:
1. Click gear icon (dashboard settings)
2. Click "Variables" tab
3. Click "Create variable"
4. **Name**: instance
5. **Label**: Instance
6. **Type**: Query
7. **Data source**: Prometheus
8. **Query**: `label_values(up, instance)`
9. Click "Add"

**Explanation**: 
- Variables enable dynamic filtering
- Query type fetches values from data source
- Label shows friendly name in UI
- `label_values()` function gets label values

**Expected**: Dropdown appears top of dashboard showing instances

---

## Solution 7: Use Variable in Panel Query
**Steps**:
1. Open panel editor (click panel → gear)
2. In Query field, edit metric:
   ```
   up{instance="$instance"}
   ```
3. Click "Apply"
4. Test: Change dropdown selection

**Explanation**: 
- `$variable` syntax substitutes variable value
- Query updates dynamically when variable changes
- Enables single dashboard to filter all panels
- Variable name is case-sensitive

**Expected**: Panels update when dropdown selection changes

---

## Solution 8: Add Table Panel
**Panel Configuration**:
```
Title: Target Status
Data source: Prometheus
Query: up (instant metric)
Visualization: Table
Columns: 
  - Instance (from labels)
  - Job (from labels)
  - Value (metric value)
```

**Explanation**: 
- Table visualization shows data in rows/columns
- Can display labels and values
- Good for list-like data
- Supports sorting and column formatting

**Expected**: Table with rows for each target

---

## Solution 9: Configure Panel Legend
**Steps**:
1. Open panel editor
2. Click "Options" tab
3. Find "Legend" section
4. Enable: "Show legend"
5. Values: Select "Last", "Min", "Max"
6. Placement: "Bottom" or "Right"
7. Click "Apply"

**Explanation**: 
- Legend shows metric series names
- Values show calculated stats (last, min, max)
- Placement controls legend position
- Useful for multi-series graphs

**Expected**: Legend appears with series information

---

## Solution 10: Export Dashboard as JSON
**Steps**:
1. Open dashboard
2. Click gear icon (dashboard settings)
3. Scroll down → "Share dashboard" section
4. Click "Export" tab
5. "Save to file" button downloads JSON

**Alternative - Copy JSON**:
```bash
# Via API
curl -H "Authorization: Bearer TOKEN" \
  http://localhost:3000/api/dashboards/uid/DASHBOARD-UID | jq '.dashboard'
```

**Explanation**: 
- JSON contains complete dashboard definition
- Useful for version control
- Can import to other Grafana instances
- Includes all panels, variables, settings

**Expected**: JSON file with dashboard configuration downloads

---

**All Solutions Complete ✅**
