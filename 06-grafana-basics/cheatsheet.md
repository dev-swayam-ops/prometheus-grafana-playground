# Cheatsheet: Grafana Basics

## Grafana Docker Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `docker run -d -p 3000:3000 grafana/grafana` | Start Grafana | `docker run -d --name grafana -p 3000:3000 grafana/grafana:latest` |
| `docker ps \| grep grafana` | Check running | `docker ps \| grep grafana` |
| `docker logs grafana` | View logs | `docker logs grafana -f` (follow logs) |
| `docker stop grafana` | Stop container | `docker stop grafana` |
| `docker rm grafana` | Remove container | `docker rm grafana` |
| `docker exec grafana <cmd>` | Run command | `docker exec grafana grafana-cli admin list-users` |

## Grafana API Endpoints

| Endpoint | Method | Purpose | Example |
|----------|--------|---------|---------|
| `/api/health` | GET | Server health | `curl http://localhost:3000/api/health` |
| `/api/org` | GET | Current org | `curl http://localhost:3000/api/org` |
| `/api/user` | GET | Current user | `curl http://localhost:3000/api/user` |
| `/api/datasources` | GET | List data sources | `curl http://localhost:3000/api/datasources` |
| `/api/dashboards/search` | GET | Search dashboards | `curl http://localhost:3000/api/search?query=prod` |
| `/api/dashboards/uid/<uid>` | GET | Get dashboard | Get dashboard JSON |
| `/api/folders` | GET | List folders | List all folders |
| `/api/teams` | GET | List teams | View teams in org |

## Default Credentials

| Field | Value |
|-------|-------|
| **Username** | admin |
| **Password** | admin |
| **Port** | 3000 |
| **Default Org** | Main Org. |

## Data Source Configuration

| Field | Value | Example |
|-------|-------|---------|
| **Name** | Display name | "Prometheus" |
| **Type** | Data source type | Prometheus, MySQL, PostgreSQL |
| **URL** | Data source address | `http://localhost:9090` |
| **Access** | Server or Browser | Server (recommended) |
| **Auth** | Authentication method | None, Basic, Bearer Token |

## Grafana Directory Structure (Container)

| Path | Purpose |
|------|---------|
| `/etc/grafana/grafana.ini` | Main configuration file |
| `/var/lib/grafana` | Data storage (sqlite database) |
| `/var/lib/grafana/plugins` | Plugin directory |
| `/var/log/grafana` | Log files |

## Docker Compose Template

```yaml
version: '3'
services:
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_INSTALL_PLUGINS=grafana-piechart-panel
    volumes:
      - grafana-storage:/var/lib/grafana
    depends_on:
      - prometheus

  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

volumes:
  grafana-storage:
```

## Common Data Sources

| Source | Type | Port | Purpose |
|--------|------|------|---------|
| **Prometheus** | prometheus | 9090 | Metrics |
| **MySQL** | mysql | 3306 | Relational data |
| **PostgreSQL** | postgres | 5432 | Relational data |
| **InfluxDB** | influxdb | 8086 | Time-series data |
| **Elasticsearch** | elasticsearch | 9200 | Logs/analytics |
| **Loki** | loki | 3100 | Log aggregation |

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+K` or `⌘K` | Open search |
| `D` | Focus dashboard |
| `?` | Show help |
| `G` | Go to dashboard |
| `S` | Save dashboard |
| `E` | Edit dashboard |

## curl Commands with Authentication

**Get API token**:
```bash
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"user":"admin","password":"admin"}'
```

**Use token for authenticated requests**:
```bash
TOKEN="eyJ0eXAiOiJKV1QiL..."
curl -H "Authorization: Bearer $TOKEN" \
  http://localhost:3000/api/dashboards/search
```

## Docker Environment Variables

| Variable | Purpose | Example |
|----------|---------|---------|
| `GF_SECURITY_ADMIN_PASSWORD` | Admin password | `admin123` |
| `GF_SECURITY_ADMIN_USER` | Admin username | `admin` |
| `GF_INSTALL_PLUGINS` | Plugins to install | `grafana-piechart-panel` |
| `GF_USERS_ALLOW_SIGN_UP` | Allow user signup | `false` |
| `GF_SERVER_ROOT_URL` | Root URL | `http://example.com` |

## Panel Query Variables

| Variable | Purpose | Example |
|----------|---------|---------|
| `$__interval` | Auto interval | `5m` (calculated) |
| `$__range` | Time range | `6h` (last 6 hours) |
| `$instance` | Custom variable | Set in templating |
| `$job` | Custom variable | Filter by job |

## Troubleshooting Commands

| Issue | Command |
|-------|---------|
| Check connectivity | `curl http://localhost:3000` |
| View errors | `docker logs grafana \| grep error` |
| Reset admin password | `docker exec grafana grafana-cli admin reset-admin-password newpass` |
| List users | `docker exec grafana grafana-cli admin list-users` |
| Check plugins | `docker exec grafana grafana-cli plugins ls` |

## Quick Setup Script

```bash
#!/bin/bash
# Start Grafana with Prometheus
docker run -d --name prometheus -p 9090:9090 prom/prometheus:latest
docker run -d --name grafana -p 3000:3000 \
  -e GF_SECURITY_ADMIN_PASSWORD=admin \
  grafana/grafana:latest

# Wait for startup
sleep 5

# Test Grafana
curl http://localhost:3000/api/health

# Test Prometheus
curl http://localhost:9090/api/v1/query?query=up
```

---

**Keep this handy while setting up Grafana!**
