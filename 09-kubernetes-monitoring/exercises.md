# Exercises: Kubernetes Monitoring

## Exercise 1: Start Minikube (Easy)
Start local Kubernetes cluster for monitoring.

**Task**:
1. Open terminal
2. Run: `minikube start --cpus=4 --memory=8192`
3. Wait for completion
4. Check: `kubectl get nodes`

**Validation**: Node status shows "Ready", no errors.

---

## Exercise 2: Add Prometheus Helm Repo (Easy)
Add Prometheus community Helm repository.

**Task**:
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm search repo kube-prometheus-stack
```

**Validation**: Search shows available chart versions.

---

## Exercise 3: Create Prometheus values.yaml (Easy)
Prepare custom Prometheus configuration.

**Task**:
Create `values.yaml`:
```yaml
prometheus:
  prometheusSpec:
    retention: 7d
    storageSpec:
      volumeClaimTemplate:
        spec:
          resources:
            requests:
              storage: 10Gi

grafana:
  adminPassword: admin123

alertmanager:
  enabled: true
```

**Validation**: File created with proper YAML syntax.

---

## Exercise 4: Install Prometheus Stack on Kubernetes (Medium)
Deploy kube-prometheus-stack using Helm.

**Task**:
```bash
kubectl create namespace monitoring
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  -f values.yaml
```

**Validation**: No errors, deployment started.

---

## Exercise 5: Verify Prometheus Pods (Medium)
Check Prometheus and related pods are running.

**Task**:
```bash
kubectl get pods -n monitoring
kubectl get svc -n monitoring
kubectl describe pod prometheus-prometheus-0 -n monitoring
```

**Validation**: Pods show status "Running", services created.

---

## Exercise 6: Port-Forward to Prometheus (Medium)
Access Prometheus running in Kubernetes.

**Task**:
```bash
kubectl port-forward -n monitoring svc/prometheus-operated 9090:9090 &
```
Then open: `http://localhost:9090`

**Validation**: Prometheus UI loads, shows targets.

---

## Exercise 7: Query Kubernetes Pod Metrics (Medium)
Query pod metrics from Kubernetes monitoring.

**Task**:
In Prometheus, run queries:
```promql
# Pod status
kube_pod_status_phase

# Node info
kube_node_info

# Pod memory usage
container_memory_usage_bytes
```

**Validation**: Queries return results with pod/node labels.

---

## Exercise 8: Port-Forward to Grafana (Medium)
Access Grafana dashboard in Kubernetes.

**Task**:
```bash
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80 &
# Login: admin / admin (or your password from values.yaml)
```

**Validation**: Grafana loads, shows dashboards.

---

## Exercise 9: Check Node and Pod Metrics (Medium)
Use kubectl to get resource metrics.

**Task**:
```bash
kubectl top nodes
kubectl top pods --all-namespaces
kubectl top pod POD_NAME -n NAMESPACE
```

**Validation**: Shows CPU and memory usage for nodes and pods.

---

## Exercise 10: View Kubernetes API Server Metrics (Medium)
Query metrics from Kubernetes API server.

**Task**:
In Prometheus, query:
```promql
# API server latency
apiserver_request_duration_seconds_bucket

# etcd requests
etcd_disk_backend_commit_duration_seconds_bucket
```

**Validation**: Metrics displayed with server/endpoint labels.

---

**All Exercises Complete ✅**
