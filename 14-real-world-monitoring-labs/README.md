# Real-world Monitoring Labs

## What You'll Learn
- Build complete monitoring for multi-tier application
- Implement SLOs and track error budgets
- Create dashboards for different roles (SRE, Dev, Manager)
- Setup incident response workflows
- Monitor complex infrastructure scenarios
- Correlate metrics across system components
- Performance tune monitoring stack

## Prerequisites
- Modules 01-13 (all previous content)
- Docker and docker-compose
- Basic application architecture understanding
- Prometheus, Grafana, AlertManager knowledge

## Key Concepts

### Multi-tier Application Stack
```
Load Balancer → API Service → Database
         ↓           ↓            ↓
      Metrics    Metrics       Metrics
         └────────→ Prometheus ←────┘
```

### Monitoring by Role
- **SRE**: SLOs, error budgets, burn rates
- **Developer**: Request latency, error rates, traces
- **Manager**: Service availability, cost metrics

### Key Metrics to Monitor
```
Application Layer:
- Request rate, latency, error rate
- Business metrics (orders, conversions)

Infrastructure Layer:
- CPU, memory, disk, network
- Container/pod health

Database Layer:
- Query latency, connection pool
- Replication lag, cache hit rate
```

## Hands-on Lab

**Objective**: Build complete monitoring for 3-tier application.

### Step 1: Start Multi-tier Application
```yaml
# docker-compose.yml
version: '3.8'
services:
  # Frontend
  nginx:
    image: nginx:latest
    ports:
      - "8080:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf

  # API Layer
  api:
    build: ./api
    environment:
      DB_HOST: mysql
      PROMETHEUS_PUSHGATEWAY: pushgateway:9091
    ports:
      - "8000:8000"

  # Database
  mysql:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: root
    ports:
      - "3306:3306"

  # Prometheus
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"

  # Grafana
  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"

  # Node Exporter
  node-exporter:
    image: prom/node-exporter
    ports:
      - "9100:9100"
```

**Start stack**:
```bash
docker-compose up -d
sleep 30
curl http://localhost:8080  # Frontend
curl http://localhost:8000/metrics  # API metrics
```

### Step 2: Configure Prometheus for All Services
```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'api'
    static_configs:
      - targets: ['api:8000']

  - job_name: 'mysql'
    static_configs:
      - targets: ['mysql:3306']

  - job_name: 'nginx'
    static_configs:
      - targets: ['nginx:9113']

  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']
```

### Step 3: Create SLO for Application
```yaml
# slo-rules.yml
groups:
  - name: slo_rules
    rules:
      # API Availability SLO: 99.9%
      - record: api:availability:ratio
        expr: up{job="api"}

      # Request latency SLO: 95% < 500ms
      - record: api:latency:p95
        expr: histogram_quantile(0.95, http_request_duration_seconds)

      # Error rate SLO: < 1%
      - record: api:error_rate:ratio
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m])) /
          sum(rate(http_requests_total[5m]))
```

### Step 4: Create Role-based Dashboards
**Dashboard 1: SRE Dashboard** (error budgets, burn rates)
**Dashboard 2: Developer Dashboard** (latency, errors by endpoint)
**Dashboard 3: Ops Dashboard** (infrastructure health)

### Step 5: Setup Alert Rules
```yaml
groups:
  - name: application_alerts
    rules:
      - alert: APIErrorRateHigh
        expr: api:error_rate:ratio > 0.01
        for: 5m
        labels:
          severity: warning

      - alert: APILatencyHigh
        expr: api:latency:p95 > 0.5
        for: 5m
        labels:
          severity: warning
```

### Step 6: Test Incident Response
```bash
# Simulate API degradation
curl -X POST http://localhost:8000/debug/slow  # Slow down API

# Monitor: Alerts fire, dashboards show degradation
# Response: Review alert, check logs, increase capacity
# Recovery: Restart service or scale up
```

## Validation
✅ All services report metrics
✅ SLO recording rules compute correctly
✅ Dashboards display for all roles
✅ Alerts fire on conditions
✅ Incident response workflow tested

## Cleanup
```bash
docker-compose down
docker volume prune
rm -rf prometheus/ grafana/
```

## Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| Incomplete instrumentation | Blind spots | Instrument all services |
| Too many dashboards | Confusion | Create role-based dashboards |
| Alert fatigue | Ignored alerts | Tune thresholds, add SLO basis |
| No runbooks | Slow response | Document playbooks |
| High cardinality | OOM | Audit and fix label values |

## Troubleshooting

**Services not scraping**:
```bash
# Check targets page
curl http://localhost:9090/targets

# Check service logs
docker logs api
docker logs mysql
```

**Dashboards not displaying data**:
- Verify prometheus.yml has correct job names
- Check data source in Grafana
- Verify scrape happening

**Alerts not firing**:
- Check alert condition queries
- Verify threshold values
- Check alert manager configuration

## Next Steps
- Deploy to production cluster
- Add distributed tracing (Jaeger)
- Implement centralized logging (ELK)
- Scale to multiple regions
- Advanced: Cost optimization, multi-tenant monitoring
