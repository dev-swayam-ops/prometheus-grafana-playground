# 07 - Grafana Dashboards and Panels

## What You'll Learn
- Dashboard creation and organization
- Panel types (graphs, gauges, tables, etc.)
- Writing advanced queries for panels
- Dashboard variables and templating
- Dashboard sharing and exporting
- Best practices for dashboard design
- Using dashboard links and annotations
- Panel data transformations

## Prerequisites
- [06-grafana-basics](../06-grafana-basics/README.md) completed
- Grafana running with Prometheus data source
- Understanding of PromQL queries
- Docker and Docker Compose installed

## Key Concepts

### Panel Types

| Type | Purpose | Best For |
|------|---------|----------|
| **Graph** | Time-series data | Trends over time |
| **Stat** | Single value | Current metrics |
| **Gauge** | Range 0-100% | Utilization metrics |
| **Table** | Tabular data | Lists of values |
| **Heatmap** | Matrix values | Distribution patterns |
| **Pie Chart** | Parts of whole | Percentage breakdown |
| **Bar Chart** | Category comparison | Comparing values |
| **Histogram** | Distribution | Frequency distribution |

### Dashboard Variables
Variables enable dynamic filtering:
```
Templating → Variables → Filter panels
Example: $instance, $job, $env
```

**Types**:
- Query: Get values from data source
- Custom: Manual values
- Ad hoc: User-defined filters

### Dashboard JSON
Dashboards are stored as JSON:
```json
{
  "dashboard": {
    "title": "My Dashboard",
    "panels": [...],
    "templating": {...}
  }
}
```

## Hands-on Lab: Build Advanced Dashboard

### Step 1: Create New Dashboard
1. Click "+" → "Dashboard"
2. Click "Add panel"
3. Set title: "System Metrics"

### Step 2: Add First Panel (CPU Usage)
**Settings**:
```
Title: CPU Usage
Metric: 100 * (1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m])))
Visualization: Graph
Unit: Percent (0-100)
Min: 0, Max: 100
```

### Step 3: Add Second Panel (Memory Usage)
**Settings**:
```
Title: Memory Available
Metric: node_memory_MemAvailable_bytes / 1024 / 1024 / 1024
Visualization: Gauge
Unit: Gigabytes
Decimals: 1
```

### Step 4: Add Third Panel (Network Throughput)
**Settings**:
```
Title: Network Receive Rate
Metric: rate(node_network_receive_bytes_total[5m]) / 1024 / 1024
Visualization: Graph
Unit: MB/s
Legend: Per interface
```

### Step 5: Add Table Panel (Target Status)
**Settings**:
```
Title: Target Status
Query: up (instant query)
Visualization: Table
Columns: job, instance, value
```

### Step 6: Add Dashboard Variable
1. Click "Dashboard settings" (gear icon)
2. Click "Variables" tab
3. Click "Add variable"
4. **Name**: instance
5. **Label**: Instance
6. **Type**: Query
7. **Data source**: Prometheus
8. **Query**: `label_values(up, instance)`
9. Click "Add"

### Step 7: Use Variable in Panels
Edit a panel:
1. Query: `up{instance="$instance"}`
2. Save

**Result**: Dropdown to filter by instance

### Step 8: Save Dashboard
1. Click "Save" button (top right)
2. Enter title: "System Overview"
3. Click "Save"

**Expected**: Dashboard saved message

## Validation

Verify dashboard:
```bash
# Get dashboard by UID
curl -H "Authorization: Bearer TOKEN" \
  http://localhost:3000/api/dashboards/uid/dashboard-uid

# List dashboards
curl -H "Authorization: Bearer TOKEN" \
  http://localhost:3000/api/search?query=System
```

## Cleanup

To delete dashboard:
1. Open dashboard
2. Click "Dashboard settings" (gear icon)
3. Click "Delete dashboard"

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| "No data" in panel | Check query works in Prometheus first |
| Variable not filtering | Use `$variable` in metric query |
| Panel shows wrong data | Verify data source selected in query |
| Legend too crowded | Use `hide_labels` or reduce series |
| Gauge shows 0% | Check metric and unit conversion |

## Troubleshooting

**Issue**: Panel query returns no data
```promql
# Test query in Prometheus
up{instance="localhost:9100"}

# Check metric exists
SHOW METRICS (if using label values)
```

**Issue**: Variable dropdown empty
```bash
# Test variable query
curl -G 'http://localhost:9090/api/v1/query' \
  --data-urlencode 'query=label_values(up, instance)'
```

**Issue**: Dashboard changes not saving
```bash
# Check Grafana permissions
# Verify user has Editor role

# Check logs
docker logs grafana | grep -i error
```

## Next Steps

✅ Dashboards and panels mastered!

**Ready to proceed to**:
- [08-grafana-alerting](../08-grafana-alerting/README.md) - Alert from Grafana dashboards
- [exercises](./exercises.md) - Practice with hands-on exercises
