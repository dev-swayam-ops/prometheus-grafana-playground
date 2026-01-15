# Troubleshooting and Debugging

## What You'll Learn
- Common Prometheus issues and solutions
- Debug missing metrics and targets
- Performance troubleshooting
- Log analysis and correlation
- Incident response procedures
- Tools for diagnosis

## Prerequisites
- Module 01: Prometheus Basics
- Module 02: Metrics and Exporters
- Module 06: Grafana Basics
- Basic command-line skills

## Key Concepts

### Common Issues
1. **Target Down** - Exporter not responding
2. **Missing Metrics** - Exporter running but no data
3. **High Cardinality** - Too many time series
4. **Slow Dashboards** - Heavy queries
5. **Alerts Not Firing** - Rule conditions not met
6. **Storage Issues** - Retention/size problems

### Troubleshooting Flow
```
Issue: Metric missing
↓
Check: Target status UP
  ✓ UP → Check metric availability
  ✗ DOWN → Check exporter/networking
↓
Check: Metrics endpoint accessible
  ✓ Yes → Check Prometheus logs
  ✗ No → Check firewall/connectivity
↓
Solution: Fix root cause
```

### Key Tools
- `curl` - Test HTTP endpoints
- `telnet` - Test TCP connectivity
- `netstat` - Check listening ports
- Prometheus UI - Status pages
- `docker logs` - Container output
- `journalctl` - System logs

## Hands-on Lab

**Objective**: Diagnose and fix common monitoring problems.

### Step 1: Verify Target Connectivity
```bash
# Check if exporter is running
curl http://localhost:9100/metrics

# If fails, check port is listening
netstat -tlnp | grep 9100

# If not, restart exporter
docker restart node-exporter
```

### Step 2: Check Prometheus Targets Page
```bash
# Open UI
http://localhost:9090/targets

# Observe: status (UP/DOWN), last scrape time
# If DOWN: check exporter logs
docker logs node-exporter | tail -50
```

### Step 3: Debug Missing Metrics
```bash
# Verify metric in exporter output
curl http://localhost:9100/metrics | grep http_requests

# If not there: Check exporter config
# Check if instrumentation enabled in app

# Query Prometheus
curl 'http://localhost:9090/api/v1/query?query=up'
```

### Step 4: Check Prometheus Config
```bash
# View loaded config
curl http://localhost:9090/config

# Validate syntax (in config file)
prometheus --config.file=prometheus.yml --dry-run

# Reload config
curl -X POST http://localhost:9090/-/reload
```

### Step 5: Analyze Query Performance
```bash
# Check query stats in Prometheus logs
docker logs prometheus | grep "slow query"

# Use HTTP slow query endpoint
curl 'http://localhost:9090/api/v1/query_range?query=up&start=...&step=60s&_debug=true'

# Optimize: Use recording rules
```

### Step 6: Check Storage and Retention
```bash
# Check Prometheus TSDB size
du -sh /prometheus/

# Check retention settings
curl http://localhost:9090/api/v1/status/config | jq '.data.retention'

# Adjust retention in config
--storage.tsdb.retention.time=30d
```

## Validation
✅ Targets show UP status in /targets page
✅ Metrics available in Prometheus queries
✅ Alerts firing correctly with expected conditions
✅ Dashboards load in < 3 seconds
✅ No ERROR entries in Prometheus logs

## Cleanup
```bash
# Clean up test targets
# Remove debug queries from dashboard
# Archive old dashboards
```

## Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| Wrong port | Target DOWN | Check config: port_number |
| Firewall blocked | Cannot connect | Open port: firewall-cmd |
| Bad scrape interval | Gaps in data | Increase interval, check exporter |
| Config syntax error | Service won't start | Run prometheus --dry-run |
| Missing labels | Cannot query | Add labels in scrape config |
| High cardinality | OOM crash | Reduce label values |

## Troubleshooting

**Target showing DOWN**:
```bash
# Test connectivity
curl -v http://target:9100/metrics

# Check firewall
netstat -tlnp | grep 9100

# Check target process
ps aux | grep exporter

# Check exporter logs
docker logs <exporter-container>
```

**Metrics missing in Prometheus**:
```bash
# Verify metrics in exporter
curl http://localhost:9100/metrics | grep metric_name

# Check scrape config
curl http://localhost:9090/config | grep scrape

# Query recent data
curl 'http://localhost:9090/api/v1/query_range?query=up&start=...'
```

**Slow dashboards**:
```bash
# Check query complexity
# View slow query logs
docker logs prometheus | grep "slow"

# Use recording rules for pre-computation
# Reduce dashboard time range
# Simplify dashboard panels
```

## Next Steps
- Module 14: Real-world Monitoring Labs
- Advanced: Distributed tracing, centralized logging
- Production: Hardening, security, scale
