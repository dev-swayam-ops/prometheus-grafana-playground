# Solutions: Kubernetes Monitoring

## Solution 1: Start Minikube

**Steps**:
```bash
minikube start --cpus=4 --memory=8192 --disk-size=30g
```

**Expected Output**:
```
Starting control plane node minikube in cluster minikube
Pulling base image ...
Creating hyperv machine ...
Starting minikube 
Done!
```

**Verification**:
```bash
kubectl get nodes
```

**Output**:
```
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   2m    v1.27.4
```

**Explanation**: 
- Starts single-node Kubernetes cluster
- 4 CPUs and 8GB memory for monitoring stack
- `kubectl get nodes` confirms cluster is running

---

## Solution 2: Add Prometheus Helm Repo

**Steps**:
```bash
helm repo add prometheus-community \
  https://prometheus-community.github.io/helm-charts
helm repo update
helm search repo kube-prometheus-stack
```

**Expected Output**:
```
NAME                            CHART VERSION   APP VERSION
prometheus-community/kube-prometheus-stack   54.0.0      v0.66.0
prometheus-community/prometheus   25.0.0
prometheus-community/alertmanager 1.0.0
```

**Explanation**: 
- Helm is package manager for Kubernetes
- Adds Prometheus community charts
- `helm search` shows available versions
- kube-prometheus-stack includes Prometheus, Grafana, AlertManager

---

## Solution 3: Create values.yaml

**File: values.yaml**:
```yaml
prometheus:
  prometheusSpec:
    retention: 7d
    storageSpec:
      volumeClaimTemplate:
        spec:
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 10Gi

grafana:
  adminPassword: admin123
  persistence:
    enabled: true
    size: 5Gi

alertmanager:
  enabled: true
  alertmanagerSpec:
    storage:
      volumeClaimTemplate:
        spec:
          resources:
            requests:
              storage: 5Gi
```

**Explanation**: 
- Configures Prometheus retention (7 days)
- Sets Grafana admin password
- Enables persistent storage
- Configures AlertManager

---

## Solution 4: Install Prometheus Stack

**Steps**:
```bash
kubectl create namespace monitoring
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  -f values.yaml \
  --wait
```

**Expected Output**:
```
NAME: prometheus
LAST DEPLOYED: 2024-01-15
NAMESPACE: monitoring
STATUS: deployed
REVISION: 1
```

**Verification**:
```bash
kubectl get all -n monitoring
```

**Explanation**: 
- Creates monitoring namespace
- Installs Prometheus operator and stack
- `--wait` waits for pods to be ready
- Takes 1-2 minutes for all components

---

## Solution 5: Verify Prometheus Pods

**Steps**:
```bash
kubectl get pods -n monitoring
kubectl get svc -n monitoring
kubectl describe pod prometheus-prometheus-0 -n monitoring
```

**Expected Output**:
```
NAME                                      READY   STATUS    RESTARTS
alertmanager-prometheus-alertmanager-0    1/1     Running   0
prometheus-prometheus-0                   2/2     Running   0
prometheus-operator-567d9f8d9f-abc12      1/1     Running   0
prometheus-grafana-abc123-def45           1/1     Running   0
prometheus-kube-state-metrics-abc123      1/1     Running   0
prometheus-node-exporter-abc12            1/1     Running   0
```

**Explanation**: 
- prometheus-prometheus-0: Main Prometheus server
- alertmanager: Alert routing
- grafana: Visualization
- kube-state-metrics: Kubernetes object metrics
- node-exporter: Host metrics

---

## Solution 6: Port-Forward to Prometheus

**Steps**:
```bash
kubectl port-forward -n monitoring svc/prometheus-operated 9090:9090 &
```

**Output**:
```
Forwarding from 127.0.0.1:9090 -> 9090
Forwarding from [::1]:9090 -> 9090
```

**Access**: `http://localhost:9090`

**Explanation**: 
- Port-forward maps local port to service
- Can access Prometheus Web UI
- `&` runs in background
- Kill with: `kill %1`

---

## Solution 7: Query Kubernetes Pod Metrics

**In Prometheus Web UI**:

**Query 1: Pod Status**:
```promql
kube_pod_status_phase
```

**Output**:
```
kube_pod_status_phase{namespace="monitoring", pod="prometheus-prometheus-0", phase="running"} 1
kube_pod_status_phase{namespace="monitoring", pod="grafana-...", phase="running"} 1
```

**Query 2: Node Information**:
```promql
kube_node_info
```

**Query 3: Pod Memory**:
```promql
container_memory_usage_bytes{pod=~".*"}
```

**Explanation**: 
- kube_pod_status_phase: Pod state (Running, Pending, Failed)
- kube_node_info: Node details
- container_memory_usage_bytes: Memory by container

---

## Solution 8: Port-Forward to Grafana

**Steps**:
```bash
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80 &
```

**Access**: `http://localhost:3000`

**Login**:
- Username: `admin`
- Password: `admin123` (from values.yaml)

**First Time**:
- Dashboard shows pre-configured Kubernetes dashboards
- "Kubernetes Cluster Monitoring" (node, pod, container metrics)

**Explanation**: 
- Grafana already has Prometheus data source configured
- Pre-built dashboards for K8s monitoring
- Can customize or create new dashboards

---

## Solution 9: Check Node and Pod Metrics

**Steps**:
```bash
kubectl top nodes
kubectl top pods --all-namespaces
kubectl top pod prometheus-prometheus-0 -n monitoring
```

**Expected Output**:
```
NAME       CPU(cores)   MEMORY(Mi)
minikube   487m         2048Mi

NAMESPACE     NAME                      CPU(cores)   MEMORY(Mi)
monitoring    prometheus-prometheus-0   150m         1024Mi
monitoring    prometheus-grafana-...    50m          256Mi
kube-system   coredns-...               10m          32Mi
```

**Explanation**: 
- `kubectl top` shows current resource usage
- CPU: cores (1000m = 1 core)
- Memory: MB
- Requires metrics-server (included with minikube)

---

## Solution 10: View API Server Metrics

**In Prometheus, run**:

**Query 1: API Server Requests**:
```promql
rate(apiserver_request_total[5m])
```

**Query 2: API Server Latency**:
```promql
apiserver_request_duration_seconds_bucket{le="1"}
```

**Query 3: etcd Disk I/O**:
```promql
rate(etcd_disk_backend_commit_duration_seconds_bucket[5m])
```

**Expected Output**:
```
apiserver_request_total{method="GET", verb="GET", endpoint="..."} 150
apiserver_request_duration_seconds_bucket{endpoint="...", le="0.1"} 500
```

**Explanation**: 
- API server metrics show request patterns
- Latency buckets help identify slow requests
- etcd metrics show cluster state store performance
- Metrics scraped from `kubernetes` service in kube-system namespace

---

**All Solutions Complete ✅**
