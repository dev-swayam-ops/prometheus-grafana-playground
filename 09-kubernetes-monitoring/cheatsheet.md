# Cheatsheet: Kubernetes Monitoring

## Kubernetes Monitoring Components

| Component | Port | Metrics Provided | Source |
|-----------|------|------------------|--------|
| **Kubelet** | 10250 | Pod/container metrics | Each node |
| **API Server** | 6443 | Request latency, etcd performance | Control plane |
| **kube-state-metrics** | 8080 | Object state (pods, nodes, deployments) | Deployment |
| **node-exporter** | 9100 | Host metrics (CPU, disk, network) | Each node |
| **etcd** | 2379 | Cluster state, consensus | Control plane |

## Common Kubernetes Metrics

| Metric | Description | Query Example |
|--------|-------------|-----------------|
| `kube_pod_status_phase` | Pod state (Running, Pending, Failed) | `kube_pod_status_phase{phase="Running"}` |
| `kube_pod_info` | Pod metadata (image, labels) | `kube_pod_info` |
| `kube_node_status_condition` | Node conditions (Ready, MemoryPressure) | `kube_node_status_condition{condition="Ready"}` |
| `container_memory_usage_bytes` | Container memory usage | `container_memory_usage_bytes` |
| `container_cpu_usage_seconds_total` | Container CPU usage | `rate(container_cpu_usage_seconds_total[5m])` |
| `kube_deployment_status_replicas` | Deployment replica status | `kube_deployment_status_replicas` |
| `apiserver_request_duration_seconds` | API server latency | `apiserver_request_duration_seconds_bucket` |

## kubectl Commands Reference

| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl get nodes` | List all nodes | `kubectl get nodes` |
| `kubectl get pods` | List pods (current namespace) | `kubectl get pods -n monitoring` |
| `kubectl describe pod` | Pod details | `kubectl describe pod POD_NAME -n NS` |
| `kubectl logs` | Pod logs | `kubectl logs POD_NAME -n NS -f` |
| `kubectl exec` | Execute command in pod | `kubectl exec POD_NAME -n NS -- bash` |
| `kubectl top nodes` | Node resource usage | `kubectl top nodes` |
| `kubectl top pods` | Pod resource usage | `kubectl top pods --all-namespaces` |
| `kubectl port-forward` | Access pod/service | `kubectl port-forward svc/NAME 9090:9090 -n NS` |

## Helm Commands for Monitoring

| Command | Purpose |
|---------|---------|
| `helm repo add <name> <url>` | Add chart repository |
| `helm repo update` | Update repo cache |
| `helm search repo <chart>` | Search available charts |
| `helm install <release> <chart>` | Install helm chart |
| `helm list -n <namespace>` | List releases |
| `helm upgrade <release> <chart>` | Update release |
| `helm uninstall <release>` | Remove release |
| `helm get values <release>` | View current values |

## Minikube Setup Commands

| Command | Purpose |
|---------|---------|
| `minikube start --cpus=4 --memory=8192` | Start cluster with resources |
| `minikube stop` | Stop cluster |
| `minikube delete` | Remove cluster |
| `minikube status` | Check cluster status |
| `minikube logs` | View cluster logs |
| `minikube addons list` | Show available addons |
| `minikube addons enable metrics-server` | Enable metrics |

## Prometheus Operator CRDs

| Resource | Purpose | Example |
|----------|---------|---------|
| **PrometheusRule** | Alert and recording rules | Define alerting rules |
| **ServiceMonitor** | Tell Prometheus to scrape service | Auto-discovery of targets |
| **PodMonitor** | Tell Prometheus to scrape pods | Direct pod monitoring |
| **Prometheus** | Prometheus deployment | Managed by operator |
| **Alertmanager** | AlertManager deployment | Routing and grouping |

## Prometheus Stack Namespace Resources

| Resource | Name | Purpose |
|----------|------|---------|
| Pod | `prometheus-prometheus-0` | Main Prometheus server |
| Pod | `prometheus-operator-...` | Prometheus operator |
| Pod | `alertmanager-prometheus-alertmanager-0` | Alert management |
| Pod | `prometheus-grafana-...` | Visualization |
| Pod | `prometheus-kube-state-metrics-...` | K8s object metrics |
| Pod | `prometheus-node-exporter-...` | Host metrics |
| Service | `prometheus-operated` | Prometheus API |
| Service | `alertmanager-operated` | AlertManager API |
| Service | `prometheus-grafana` | Grafana UI |

## Common Kubernetes Queries

**CPU Usage by Pod**:
```promql
sum(rate(container_cpu_usage_seconds_total[5m])) by (pod_name)
```

**Memory Usage by Pod**:
```promql
sum(container_memory_usage_bytes) by (pod_name)
```

**Pod Restart Count**:
```promql
kube_pod_container_status_restarts_total
```

**Node Capacity vs Usage**:
```promql
(kube_node_status_allocatable_cpu_cores) / (kube_node_status_capacity_cpu_cores)
```

**Deployment Replica Status**:
```promql
kube_deployment_status_replicas_available / kube_deployment_spec_replicas
```

**API Server Request Rate**:
```promql
rate(apiserver_request_total[5m])
```

## values.yaml Configuration

**Prometheus Settings**:
```yaml
prometheus:
  prometheusSpec:
    retention: 7d              # Keep metrics for 7 days
    scrapeInterval: 30s        # Scrape every 30 seconds
    evaluationInterval: 30s    # Evaluate rules every 30 seconds
    storageSpec:
      volumeClaimTemplate:
        spec:
          resources:
            requests:
              storage: 50Gi
```

**Grafana Settings**:
```yaml
grafana:
  adminPassword: admin123
  persistence:
    enabled: true
    size: 5Gi
  datasources:
    datasources.yaml:
      apiVersion: 1
      datasources:
      - name: Prometheus
        type: prometheus
        url: http://prometheus-operated:9090
```

## Port-Forward Reference

**Prometheus**:
```bash
kubectl port-forward -n monitoring svc/prometheus-operated 9090:9090
# Access: http://localhost:9090
```

**Grafana**:
```bash
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
# Access: http://localhost:3000 (admin/admin)
```

**AlertManager**:
```bash
kubectl port-forward -n monitoring svc/alertmanager-operated 9093:9093
# Access: http://localhost:9093
```

## Troubleshooting Commands

| Issue | Command |
|-------|---------|
| Pod logs | `kubectl logs POD_NAME -n monitoring -f` |
| Pod events | `kubectl describe pod POD_NAME -n monitoring` |
| Pod metrics | `kubectl top pod POD_NAME -n monitoring` |
| Node metrics | `kubectl top nodes` |
| Prometheus targets | Visit `http://localhost:9090/targets` |
| Prometheus rules | Visit `http://localhost:9090/rules` |
| Grafana datasource | Check `Configuration → Datasources` |
| Check scrape config | `kubectl get servicemonitor -n monitoring` |
| Check alerts | `kubectl get prometheusrule -n monitoring` |

## Monitoring Stack Helm Installation

**Quick Start**:
```bash
# 1. Create namespace
kubectl create namespace monitoring

# 2. Add and update repo
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# 3. Install stack
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  -f values.yaml \
  --wait

# 4. Port-forward
kubectl port-forward -n monitoring svc/prometheus-operated 9090:9090 &
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80 &

# 5. Access
# Prometheus: http://localhost:9090
# Grafana: http://localhost:3000
```

**Cleanup**:
```bash
helm uninstall prometheus --namespace monitoring
kubectl delete namespace monitoring
```

## Best Practices

✅ **DO**:
- Set appropriate retention periods
- Use ServiceMonitor for auto-discovery
- Enable persistent storage
- Monitor the monitoring stack itself
- Set resource requests/limits
- Use namespaces for isolation
- Regularly back up Prometheus data
- Test alerts in staging first

❌ **DON'T**:
- Forget to set storage limits
- Monitor too many metrics
- Expose Prometheus/Grafana publicly
- Use default credentials in production
- Scrape without rate limiting
- Alert on everything
- Ignore etcd performance
- Run without persistence

---

**Keep this open during K8s monitoring setup!**
