# 09 - Kubernetes Monitoring

## What You'll Learn
- Kubernetes monitoring architecture
- Deploying Prometheus in Kubernetes
- Using kube-state-metrics for cluster monitoring
- Monitoring pod, node, and cluster metrics
- Service discovery in Kubernetes
- Scraping kubelet and API server metrics
- Setting up Grafana dashboards for Kubernetes
- Monitoring best practices for K8s

## Prerequisites
- [08-grafana-alerting](../08-grafana-alerting/README.md) completed
- Kubernetes cluster (minikube, kind, or cloud)
- kubectl installed and configured
- Understanding of Prometheus and Grafana
- Basic Kubernetes knowledge

## Key Concepts

### Kubernetes Monitoring Architecture

```
Kubelet (metrics) ← Prometheus scrapes
  ↓
API Server (metrics)
  ↓
kube-state-metrics → Metrics about objects (Pods, Nodes, Deployments)
  ↓
Prometheus → Grafana dashboards
```

### Kubernetes Metrics Sources

| Component | Port | Metrics |
|-----------|------|---------|
| **Kubelet** | 10250 | Container, node metrics |
| **API Server** | 6443 | API request metrics |
| **Controller Manager** | 10252 | Reconciliation metrics |
| **Scheduler** | 10251 | Scheduling metrics |
| **kube-state-metrics** | 8080 | Object state (Pods, Nodes, etc.) |
| **node-exporter** | 9100 | Host-level metrics |

### Service Discovery

Kubernetes has native service discovery:
```yaml
scrape_configs:
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
```

### Common Kubernetes Metrics

| Metric | Purpose |
|--------|---------|
| `kube_pod_status_phase` | Pod running/pending/failed |
| `kube_pod_container_resource_requests_cpu_cores` | CPU requests |
| `kube_pod_container_resource_limits_memory_bytes` | Memory limits |
| `kube_node_status_condition` | Node health (Ready, MemoryPressure) |
| `container_memory_usage_bytes` | Container memory usage |
| `container_cpu_usage_seconds_total` | Container CPU time |
| `kube_deployment_status_replicas_available` | Replicas available |

## Hands-on Lab: Monitor Local Kubernetes

### Step 1: Start Local Kubernetes (minikube)
```bash
minikube start --cpus=4 --memory=4096
minikube addons enable metrics-server
```

**Expected Output**:
```
✓ Cluster created successfully!
✓ Enabled metrics-server
```

### Step 2: Add Prometheus Helm Chart
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace \
  --values values.yaml
```

**values.yaml**:
```yaml
prometheus:
  prometheusSpec:
    retention: 15d
    storageSpec:
      volumeClaimTemplate:
        spec:
          resources:
            requests:
              storage: 10Gi

grafana:
  adminPassword: admin
  service:
    type: NodePort

kubeStateMetrics:
  enabled: true

nodeExporter:
  enabled: true
```

### Step 3: Verify Deployment
```bash
kubectl get pods -n monitoring
kubectl get svc -n monitoring
```

**Expected Output**:
```
NAME                                 READY   STATUS    RESTARTS
prometheus-kube-prometheus-operator   1/1     Running   0
prometheus-grafana                    1/1     Running   0
prometheus-kube-state-metrics         1/1     Running   0
prometheus-node-exporter-xxxxx        1/1     Running   0
```

### Step 4: Access Prometheus
```bash
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090
```

Open: `http://localhost:9090`

### Step 5: Access Grafana
```bash
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
```

Open: `http://localhost:3000` (admin/prom-operator)

### Step 6: Explore Kubernetes Metrics
In Prometheus Graph tab:

```promql
# Pod status
kube_pod_status_phase

# Node status
kube_node_status_condition

# Container CPU usage
rate(container_cpu_usage_seconds_total[5m])

# Container memory
container_memory_usage_bytes
```

### Step 7: Import Pre-built Dashboards
In Grafana:
1. Click "+" → "Import"
2. Dashboard ID: `6417` (K8s Cluster Monitoring)
3. Select Prometheus data source
4. Click "Import"

### Step 8: Verify Pod Monitoring
```bash
# Query pod metrics
curl -G 'http://localhost:9090/api/v1/query' \
  --data-urlencode 'query=kube_pod_info'
```

**Expected**: JSON with pod information

## Validation

Verify Kubernetes monitoring:
```bash
# Check all monitoring components
kubectl get all -n monitoring

# Check for scrape targets
curl -s http://localhost:9090/api/v1/targets | jq '.data.activeTargets | length'

# Check node metrics
kubectl top nodes

# Check pod metrics
kubectl top pods --all-namespaces
```

## Cleanup

```bash
helm uninstall prometheus -n monitoring
kubectl delete namespace monitoring
minikube stop
```

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| kube-state-metrics not scraping | Verify service discovery config and labels |
| "No data" for pod metrics | Check pod has monitoring label annotation |
| High memory usage | Increase retention period or add storage |
| Missing node metrics | Ensure node-exporter DaemonSet running |
| Can't access Prometheus | Use port-forward, not LoadBalancer |

## Troubleshooting

**Issue**: Pods not appearing in metrics
```bash
# Check kube-state-metrics running
kubectl get pods -n monitoring | grep kube-state-metrics

# Check logs
kubectl logs -n monitoring -l app=kube-state-metrics -f

# Query kube-state-metrics
curl http://kube-state-metrics:8080/metrics
```

**Issue**: Node metrics missing
```bash
# Check node-exporter DaemonSet
kubectl get ds -n monitoring

# Check node-exporter running on all nodes
kubectl get pods -n monitoring -o wide | grep node-exporter
```

**Issue**: High memory consumption
```bash
# Check Prometheus retention
kubectl get prometheus -n monitoring -o yaml | grep retention

# Reduce retention
kubectl patch prometheus -n monitoring --type merge \
  -p '{"spec":{"retention":"7d"}}'
```

## Next Steps

✅ Kubernetes monitoring configured!

**Ready to proceed to**:
- [10-service-discovery-and-scraping](../10-service-discovery-and-scraping/README.md) - Advanced service discovery
- [exercises](./exercises.md) - Practice with hands-on exercises
