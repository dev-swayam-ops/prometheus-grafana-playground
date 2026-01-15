# Exercises: Troubleshooting and Debugging

## Exercise 1: Check Target Status (Easy)
View target health in Prometheus.

**Task**:
1. Open: `http://localhost:9090/targets`
2. Look at each target
3. Note: status (UP/DOWN), last scrape, scrape duration
4. Document any DOWN targets

**Validation**: Can identify target status and reason.

---

## Exercise 2: Test Target Connectivity (Easy)
Verify exporter is responding.

**Task**:
```bash
# Test main target
curl -I http://localhost:9100/metrics

# Should return: 200 OK
# If fails: target is down
```

Test 3 different targets.

**Validation**: Can test connectivity and identify issues.

---

## Exercise 3: Query Prometheus Metrics (Easy)
Verify metrics are being scraped.

**Task**:
1. Open Prometheus: `http://localhost:9090`
2. Query: `up`
3. Should show: status=UP targets
4. Query: `rate(up[5m])`
5. Check results

**Validation**: Understand query results.

---

## Exercise 4: View Prometheus Logs (Easy)
Check Prometheus output for errors.

**Task**:
```bash
# For Docker
docker logs prometheus | tail -50

# For systemd
journalctl -u prometheus -n 50

# Look for: ERROR, WARN entries
```

Document any issues found.

**Validation**: Can read and interpret logs.

---

## Exercise 5: Check Scrape Config (Medium)
Review how Prometheus scrapes targets.

**Task**:
```bash
# View loaded config
curl http://localhost:9090/config | jq '.data'

# Verify: job_name, targets, scrape_interval
# Check: relabel_configs, metric_relabel
```

Document one scrape config.

**Validation**: Understand scrape configuration.

---

## Exercise 6: Test Exporter Metrics Output (Medium)
Verify raw metrics from exporter.

**Task**:
```bash
# Get raw metrics
curl http://localhost:9100/metrics

# Look for: TYPE, HELP, metric data
# Find 5 different metric types
# Check metric names follow convention
```

Document findings.

**Validation**: Can read and interpret metric format.

---

## Exercise 7: Reload Prometheus Config (Medium)
Apply config changes without restart.

**Task**:
1. Edit prometheus.yml (add new target)
2. Validate syntax: `prometheus --config.file=... --dry-run`
3. Reload: `curl -X POST http://localhost:9090/-/reload`
4. Verify in targets page

**Validation**: New target appears within 1 minute.

---

## Exercise 8: Investigate Missing Metric (Medium)
Debug why metric isn't appearing.

**Task**:
1. Pick a metric: `http_requests_total`
2. Check exporter output: `curl ... | grep http_requests`
3. Check Prometheus: Query the metric
4. If missing: Check logs
5. Document resolution steps

**Validation**: Can trace metric from source to Prometheus.

---

## Exercise 9: Analyze Query Performance (Medium)
Check dashboard query speed.

**Task**:
1. Open Grafana dashboard
2. Open browser developer tools (F12)
3. Reload dashboard
4. Check Network tab for query times
5. Identify slow queries (> 5s)

**Validation**: Can identify slow queries and propose optimization.

---

## Exercise 10: Check Storage and Retention (Medium)
Verify Prometheus storage health.

**Task**:
```bash
# Check disk usage
du -sh /prometheus/

# Query retention
curl http://localhost:9090/api/v1/status/config

# Calculate: based on metrics/retention
# Document findings
```

**Validation**: Understand storage impact and planning.

---

**All Exercises Complete ✅**
