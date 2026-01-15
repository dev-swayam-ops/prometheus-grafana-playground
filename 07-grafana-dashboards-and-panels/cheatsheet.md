# Cheatsheet: Grafana Dashboards and Panels

## Panel Types and Use Cases

| Panel Type | Use Case | Data Format | Example |
|-----------|----------|------------|---------|
| **Graph** | Time-series trends | Multi-series over time | CPU, Memory over hours |
| **Stat** | Single current value | Scalar value | Current error rate |
| **Gauge** | 0-100% utilization | Single value 0-100 | Memory %, CPU % |
| **Table** | Tabular data | Rows and columns | List of servers |
| **Heatmap** | Distribution pattern | Time-series matrix | Error distribution |
| **Pie Chart** | Part-of-whole | Categories with values | % breakdown |
| **Bar Chart** | Category comparison | Categories vs values | CPU per host |
| **Histogram** | Frequency distribution | Bucketed values | Latency buckets |

## Dashboard Variables

### Types

| Type | Purpose | Example |
|------|---------|---------|
| **Query** | Get values from data source | `label_values(up, instance)` |
| **Custom** | Manual list of options | `prod, staging, dev` |
| **Ad hoc** | User-defined on the fly | Filter any label |
| **Constant** | Fixed value for all | Environment name |
| **Datasource** | Select data source | Choose Prometheus vs InfluxDB |

### Variable Syntax

```
$variable           # Simple substitution
${variable}         # Alternative syntax
$__auto_interval    # Built-in auto interval
$__range            # Time range selected
```

## Panel Query Configuration

| Field | Purpose | Example |
|-------|---------|---------|
| **Data source** | Where to query | Prometheus, MySQL |
| **Query** | Metric/SQL expression | `rate(requests[5m])` |
| **Legend** | Series naming | `{{job}}-{{instance}}` |
| **Interval** | Query resolution | `5m`, `$__interval` |
| **Min interval** | Minimum resolution | `10s` |

## Common Panel Queries

| Scenario | Query |
|----------|-------|
| **CPU Usage %** | `100 * (1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m])))` |
| **Memory Usage %** | `(1 - node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes) * 100` |
| **Network Throughput** | `rate(node_network_receive_bytes_total[5m]) / 1024 / 1024` |
| **Disk Usage %** | `(1 - node_filesystem_avail_bytes/node_filesystem_size_bytes) * 100` |
| **Uptime** | `time() - node_boot_time_seconds` |
| **Error Rate** | `rate(errors_total[5m]) / rate(requests_total[5m])` |

## Panel Editor Tabs

| Tab | Purpose | Options |
|-----|---------|---------|
| **Query** | Data selection | Query expression, data source |
| **Visualization** | Chart type | Graph, Gauge, Stat, Table, etc. |
| **Field** | Number formatting | Unit, decimals, min/max |
| **Options** | Panel-specific settings | Legend, threshold, colors |

## Unit Formatting

| Unit | Conversion | Example |
|------|-----------|---------|
| **Percent** | No change | 0-100 |
| **Bytes** | Auto format | 1.5GB, 512MB |
| **MB/s** | Throughput | Network speed |
| **seconds** | Time | Duration |
| **short** | Auto units | 1.2K, 500M |

## Legend Configuration

| Setting | Purpose | Values |
|---------|---------|--------|
| **Show legend** | Display legend | true/false |
| **Legend values** | Show stats | Last, Min, Max, Mean |
| **Legend placement** | Where to show | Bottom, Right, Top |
| **Legend width** | Legend size | Auto, 200px, 300px |
| **Legend calcs** | Calculations | average, sum, count |

## Dashboard Templating

```json
{
  "templating": {
    "list": [
      {
        "name": "instance",
        "type": "query",
        "datasource": "Prometheus",
        "query": "label_values(up, instance)",
        "current": {"value": "localhost:9100"}
      }
    ]
  }
}
```

## Dashboard JSON Structure

```json
{
  "dashboard": {
    "title": "Dashboard Title",
    "panels": [
      {
        "id": 1,
        "title": "Panel Title",
        "type": "graph",
        "targets": [{"expr": "up"}]
      }
    ],
    "templating": {...},
    "time": {"from": "now-6h", "to": "now"},
    "timezone": "browser"
  }
}
```

## API Endpoints for Dashboards

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/dashboards/search` | GET | List dashboards |
| `/api/dashboards/uid/<uid>` | GET | Get dashboard |
| `/api/dashboards/db` | POST | Create dashboard |
| `/api/dashboards/uid/<uid>` | PUT | Update dashboard |
| `/api/dashboards/uid/<uid>` | DELETE | Delete dashboard |

## curl Commands

**Get dashboard**:
```bash
curl -H "Authorization: Bearer TOKEN" \
  http://localhost:3000/api/dashboards/uid/dashboard-uid
```

**Create dashboard**:
```bash
curl -X POST -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d @dashboard.json \
  http://localhost:3000/api/dashboards/db
```

**List dashboards**:
```bash
curl -H "Authorization: Bearer TOKEN" \
  http://localhost:3000/api/search?query=metrics
```

## Panel Refresh Intervals

| Interval | Meaning |
|----------|---------|
| `5s` | Every 5 seconds (updates frequently) |
| `30s` | Every 30 seconds (standard) |
| `1m` | Every minute (slower updates) |
| `5m` | Every 5 minutes (slow) |
| `1h` | Every hour (rare updates) |

## Troubleshooting Panels

| Problem | Cause | Fix |
|---------|-------|-----|
| "No data" | Query returns nothing | Verify metric exists in Prometheus |
| Blank panel | Wrong data source | Select correct data source |
| Wrong values | Incorrect query | Test query in Prometheus first |
| Slow loading | Too much data | Add time filter or reduce range |
| Misaligned scales | Incompatible units | Check unit settings |

## Dashboard Best Practices

✅ **DO**:
- Use meaningful titles
- Group related panels
- Set appropriate units
- Use consistent colors
- Document dashboard purpose
- Version control dashboards

❌ **DON'T**:
- Overcrowd dashboards (max 6-8 panels)
- Mix unrelated metrics
- Forget to set units
- Leave default colors
- Create too many dashboards

## Quick Panel Template

```yaml
Panel Configuration:
  Title: "Metric Name"
  Data source: "Prometheus"
  Query: "metric_expression"
  Visualization: "Graph"
  
Graph Options:
  Legend: Show
  Unit: Appropriate unit
  Min/Max: Set range
  Decimals: 2

Refresh: "30s"
```

---

**Keep this handy while building dashboards!**
