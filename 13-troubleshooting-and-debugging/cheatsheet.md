# Cheatsheet: Troubleshooting and Debugging

## Quick Diagnostic Commands

| Issue | Command | Expected Output |
|-------|---------|-----------------|
| **Target down** | `curl -I http://target:9100/metrics` | 200 OK |
| **Port listening** | `netstat -tlnp \| grep 9100` | LISTEN |
| **Prometheus config** | `curl http://localhost:9090/config` | JSON config |
| **All targets** | `curl http://localhost:9090/api/v1/targets` | JSON array |
| **Metric exists** | `curl http://localhost:9090/api/v1/query?query=up` | Results |

## Common Issues and Fixes

| Issue | Cause | Fix |
|-------|-------|-----|
| **Target DOWN** | Exporter crashed/unreachable | `docker restart exporter` or check firewall |
| **Metrics missing** | Not instrumented in app | Add metric instrumentation |
| **High cardinality** | Too many label values | Reduce unique label combinations |
| **Slow dashboard** | Heavy queries | Use recording rules, reduce range |
| **Alerts not firing** | Condition not met | Review alert condition, check threshold |
| **Config error** | YAML syntax | Run `prometheus --dry-run` |

## Prometheus UI Pages

| Page | URL | Purpose |
|------|-----|---------|
| Targets | `/targets` | See scrape status UP/DOWN |
| Graph | `/graph` | Query and visualize metrics |
| Config | `/config` | View loaded configuration |
| Rules | `/rules` | See alert and recording rules |
| Status | `/status` | Service health info |

## API Endpoints for Debugging

| Endpoint | Purpose |
|----------|---------|
| `/api/v1/query` | Query metric data |
| `/api/v1/query_range` | Query over time range |
| `/api/v1/targets` | List scrape targets |
| `/api/v1/labels` | List all labels |
| `/api/v1/label/__name__/values` | List metric names |
| `/api/v1/rules` | List alert rules |
| `/api/v1/status/config` | Configuration status |

## Log Analysis Commands

| Command | Purpose |
|---------|---------|
| `docker logs prometheus` | View Prometheus logs |
| `docker logs prometheus \| grep ERROR` | Find errors |
| `docker logs prometheus \| grep -i "slow"` | Find slow queries |
| `journalctl -u prometheus -n 100` | Systemd logs |
| `journalctl -u prometheus -f` | Follow logs |

## Connectivity Testing

**Test exporter port**:
```bash
# ICMP ping
ping host

# TCP port
telnet host 9100

# HTTP endpoint
curl http://host:9100/metrics
curl -I http://host:9100/metrics
```

## Config Validation

**Syntax check**:
```bash
prometheus --config.file=prometheus.yml --dry-run
```

**Reload without restart**:
```bash
curl -X POST http://localhost:9090/-/reload
```

**View actual config**:
```bash
curl http://localhost:9090/config | jq
```

## Metric Inspection

**Get raw metrics from exporter**:
```bash
curl http://target:9100/metrics | grep metric_name
```

**Find metric in Prometheus**:
```bash
curl 'http://localhost:9090/api/v1/query?query=metric_name'
```

**Check cardinality**:
```bash
# Total series
curl 'http://localhost:9090/api/v1/query?query=count(count%20by%20(__name__)(up))'

# Per metric
curl 'http://localhost:9090/api/v1/query?query=topk(10,%20count%20by%20(__name__)(up))'
```

## Query Performance

**Check slow queries**:
```bash
docker logs prometheus | grep "slow query"
```

**Query stats**:
```bash
curl 'http://localhost:9090/api/v1/query_range?query=up&start=...&end=...&step=60s&_debug=true'
```

**Optimize strategies**:
- Use recording rules for complex queries
- Reduce time range
- Increase step size
- Use aggregation to reduce cardinality

## Storage Debugging

**Check disk usage**:
```bash
du -sh /prometheus/
du -sh /prometheus/wal
du -sh /prometheus/blocks
```

**Check retention**:
```bash
curl http://localhost:9090/api/v1/status/config | jq '.data.storage'
```

**Calculate storage needs**:
```
Needed = Series Count × Retention Seconds × ~1-2 bytes/sample
```

## Troubleshooting Flowchart

```
Issue: Metric missing
├─ Check target UP
│  ├─ UP → Check scrape config
│  │  └─ Valid → Check logs for errors
│  └─ DOWN → Check connectivity
│     └─ Run: curl target:9100/metrics
├─ Check exporter has metric
│  ├─ Yes → Check prometheus config
│  └─ No → Add instrumentation
└─ Check recent data
   └─ Run: query_range with recent time
```

## Alert Debugging

**Check alert rules**:
```bash
curl http://localhost:9090/rules
```

**Test alert condition**:
```bash
# Query the condition expression
curl 'http://localhost:9090/api/v1/query?query=ALERT_CONDITION'

# If returns data, alert could fire
```

**Check alert state**:
```bash
curl http://localhost:9090/api/v1/rules | jq '.data[].rules[] | select(.name=="ALERT_NAME")'
```

## Performance Best Practices

✅ **DO**:
- Use dashboard time ranges < 7 days
- Use recording rules for common queries
- Limit cardinality with label selection
- Archive old data
- Monitor Prometheus memory usage
- Use appropriate retention periods
- Document metric purposes

❌ **DON'T**:
- Query raw high-cardinality metrics
- Use very small step sizes (< 60s)
- Run aggregations across millions of series
- Forget to validate config before reload
- Let Prometheus run out of disk space
- Query full year of data in dashboard
- Set scrape intervals < 15s

## Emergency Recovery

**Prometheus won't start**:
```bash
# Check config syntax
prometheus --config.file=prometheus.yml --dry-run

# Check disk space
df -h /prometheus

# Check permissions
ls -la /prometheus
```

**Storage full**:
```bash
# Delete old blocks
rm -rf /prometheus/blocks/[oldest-timestamp]

# Or, adjust retention
prometheus --storage.tsdb.retention.size=50GB
```

**High memory usage**:
```bash
# Identify high-cardinality metrics
curl 'http://localhost:9090/api/v1/query?query=topk(10,count%20by%20(__name__)(up))'

# Remove offending metrics from scrape config
# Restart Prometheus
```

---

**Keep this handy for troubleshooting!**
