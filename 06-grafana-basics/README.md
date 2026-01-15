# 06 - Grafana Basics

## What You'll Learn
- Grafana architecture and components
- Installing and starting Grafana
- Adding data sources (Prometheus, databases, APIs)
- Configuring user accounts and authentication
- Exploring pre-built dashboards
- Understanding dashboard variables
- Creating your first simple dashboard

## Prerequisites
- [05-alertmanager-and-notifications](../05-alertmanager-and-notifications/README.md) completed
- Prometheus with metrics running
- Docker and Docker Compose installed
- Basic understanding of monitoring concepts

## Key Concepts

### Grafana Architecture
```
Users → Grafana UI → Data Sources (Prometheus, MySQL, etc.)
                  → Dashboards (visualizations)
                  → Alerts (alerts based on rules)
                  → Users & Teams (permissions)
```

### Core Components

**1. Data Sources**: Where Grafana gets data
```
Prometheus, InfluxDB, MySQL, PostgreSQL, 
Elasticsearch, GraphQL, APIs, etc.
```

**2. Dashboards**: Collections of panels (visualizations)
```
Dashboard → Panels (graphs, gauges, tables, heatmaps)
         → Variables (dynamic filtering)
         → Annotations (events)
```

**3. Users & Orgs**: Access control
```
Organization → Teams → Users
                    → Roles (Admin, Editor, Viewer)
```

**4. Alerts**: Monitor dashboard metrics
```
Panel → Alert Rule → Notification Channels
```

### Default Credentials
```
Username: admin
Password: admin
(Change on first login for security)
```

## Hands-on Lab: Grafana Setup with Prometheus

### Step 1: Create docker-compose.yml
```yaml
version: '3'
services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./rules.yml:/etc/prometheus/rules.yml
    command:
      - --config.file=/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    depends_on:
      - prometheus
    volumes:
      - grafana-storage:/var/lib/grafana

  node-exporter:
    image: prom/node-exporter:latest
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro

volumes:
  grafana-storage:
```

### Step 2: Start Services
```bash
docker-compose up -d
docker-compose ps
```

**Expected Output**:
```
NAME              IMAGE                  STATUS
prometheus        prom/prometheus        Up 10 seconds
grafana           grafana/grafana        Up 10 seconds
node-exporter     prom/node-exporter     Up 10 seconds
```

### Step 3: Access Grafana UI
Open browser: `http://localhost:3000`

**Expected**: Grafana login page

### Step 4: Login to Grafana
```
Username: admin
Password: admin
```

**Expected**: Grafana home dashboard with welcome message

### Step 5: Change Admin Password
1. Click profile icon (top right)
2. Select "Change password"
3. Enter new password (recommended for security)

### Step 6: Add Prometheus Data Source
1. Click "Connections" (left sidebar)
2. Click "Data sources"
3. Click "Add data source"
4. Select "Prometheus"
5. Set URL: `http://prometheus:9090`
6. Click "Save & test"

**Expected**: "Data source is working" message

### Step 7: Create Simple Dashboard
1. Click "+" → "Dashboard"
2. Click "Add panel"
3. In Query section, select Prometheus
4. Enter metric: `up`
5. Click "Apply"

**Expected**: Graph showing target health metrics

### Step 8: Explore Panel Options
1. Click gear icon (top right of panel)
2. Explore tabs:
   - Query (change metric)
   - Visualization (chart type)
   - Field (formatting)
   - Options (panel settings)

## Validation

Verify setup:
```bash
# Check services running
docker-compose ps

# Access Grafana API
curl http://localhost:3000/api/health

# List data sources
curl -H "Authorization: Bearer $(curl -s -X POST \
  http://localhost:3000/api/login \
  -d 'user=admin&password=admin' | jq -r '.token')" \
  http://localhost:3000/api/datasources
```

**Expected**: All services running, API returns 200 OK, data sources listed

## Cleanup

```bash
docker-compose down
docker volume rm grafana-storage
```

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| Can't connect to Prometheus | Check URL in data source: `http://prometheus:9090` |
| "Data source is working" fails | Ensure Prometheus container is running |
| Graphs show "No data" | Verify metric exists in Prometheus |
| Password reset doesn't work | Restart Grafana: `docker-compose restart grafana` |
| Panel shows blank | Check query syntax, try simpler metric |

## Troubleshooting

**Issue**: Can't access Grafana UI
```bash
# Check if container running
docker ps | grep grafana

# Check logs
docker logs grafana

# Verify port mapping
docker port grafana
```

**Issue**: Data source test fails
```bash
# Test Prometheus connectivity from Grafana container
docker exec grafana curl http://prometheus:9090

# Check Prometheus is running
curl http://localhost:9090
```

**Issue**: Can't login with default credentials
```bash
# Reset admin password
docker exec grafana grafana-cli admin reset-admin-password newpassword

# Restart Grafana
docker-compose restart grafana
```

## Next Steps

✅ Grafana basics mastered!

**Ready to proceed to**:
- [07-grafana-dashboards-and-panels](../07-grafana-dashboards-and-panels/README.md) - Build advanced dashboards
- [exercises](./exercises.md) - Practice with hands-on exercises
