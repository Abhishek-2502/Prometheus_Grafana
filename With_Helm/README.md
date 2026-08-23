## 1. Install Docker, Kind, Kubectl
- Follow this guide to install Kind and Kubectl: [Link](https://github.com/Abhishek-2502/Docker_K8s/tree/main/K8s/KIND_Cluster_Install)

## 2. Install HELM 

- Follow this guide to install Helm: [Link](https://github.com/Abhishek-2502/Docker_K8s/tree/main/K8s/Basics_to_Advanced/Helm)


## 3. Install Kube Prometheus Stack 

### Add Helm Repositories
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```
```
helm repo add stable https://charts.helm.sh/stable
```
```
helm repo add grafana https://grafana.github.io/helm-charts
```
```
helm repo update
```
```
helm repo list
```

### Create Monitoring Namespace
```
kubectl create namespace monitoring
```

### Install the Stack
```
helm install kind-prometheus prometheus-community/kube-prometheus-stack --version 88.5.3 --namespace monitoring --set prometheus.service.nodePort=30000 --set prometheus.service.type=NodePort --set grafana.service.nodePort=31000 --set grafana.service.type=NodePort --set alertmanager.service.nodePort=32000 --set alertmanager.service.type=NodePort --set prometheus-node-exporter.service.nodePort=32001 --set prometheus-node-exporter.service.type=NodePort
```

### Install Loki
```
helm install kind-loki grafana/loki --version 7.3.0 --namespace monitoring --values loki-values.yaml
```

The values file runs Loki in single-binary mode with filesystem storage. This is suitable for a local Kind cluster; use persistent storage and a production storage backend for durable logs.

If Promtail reports HTTP `429` ingestion-rate errors while importing existing logs, update the release with the configured local-cluster limits:
```
helm upgrade kind-loki grafana/loki --version 7.3.0 --namespace monitoring --values loki-values.yaml
```

### Install Promtail log collector
Loki stores logs but does not collect them. Install Promtail to discover Kubernetes pods and send their logs to Loki:
```
helm install kind-promtail grafana/promtail --version 6.17.1 --namespace monitoring --values promtail-values.yaml
```

### Verify Installation
```
kubectl get svc -n monitoring

kubectl get namespace

helm list -n monitoring

kubectl get pods -n monitoring
```

Wait until the Loki and Grafana pods show `Running` and all containers are ready.

You should see:

- One or more `kube-state-metrics` pods. These pods query the Kubernetes API server and expose metrics about the state of Kubernetes objects, such as Pods, Deployments, Nodes, Services, ConfigMaps, Secrets, Jobs, and StatefulSets. They do not directly collect CPU, memory, disk, or network usage from the nodes.

- One `node-exporter` pod per Kubernetes node. `node-exporter` normally runs as a DaemonSet, which schedules one pod on each eligible control-plane and worker node. These pods collect host-level metrics such as CPU usage, memory usage, disk space, disk I/O, and network statistics.

For example, in a cluster with one control-plane node and two worker nodes, you may see:

- One `kube-state-metrics` pod
- Three `node-exporter` pods

`kube-state-metrics` reports the desired and current state of Kubernetes resources, while `node-exporter` reports the operating-system and hardware condition of each node. Prometheus usually scrapes metrics from both components and stores them for querying and alerting.

The exact number of pods may vary depending on the configured replicas, node taints, tolerations, and scheduling rules.

### Uninstall (if needed)
```
helm uninstall kind-prometheus -n monitoring

helm uninstall kind-loki -n monitoring

helm uninstall kind-promtail -n monitoring
```

---

## 4. Access Prometheus & Grafana

### Linux
```
kubectl port-forward svc/kind-prometheus-kube-prome-prometheus -n monitoring 9090:9090 --address=0.0.0.0 &

kubectl port-forward svc/kind-prometheus-grafana -n monitoring 31000:80 --address=0.0.0.0 &

kubectl port-forward svc/kind-loki -n monitoring 31001:3100 --address=0.0.0.0 &
```

### Windows
```
kubectl port-forward svc/kind-prometheus-kube-prome-prometheus -n monitoring 9090:9090 

kubectl port-forward svc/kind-prometheus-grafana -n monitoring 31000:80 

kubectl port-forward svc/kind-loki -n monitoring 31001:3100
```

#### Access in Browser:

 - Prometheus: http://localhost:9090

 - Grafana: http://localhost:31000

 - Loki: http://localhost:31001/ready

 - Prometheus Metrics: http://localhost:9090/metrics

Grafana Credentials
 - `username:` admin
 - `password:` prom-operator

Get Password using kubectl if above password is invalid:
```
kubectl get secret kind-prometheus-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode
```

### Add Loki to Grafana

In Grafana, add a Loki data source with this URL:
```
http://kind-loki:3100
```

Promtail sends Kubernetes pod logs to Loki automatically. Check its status with:
```
kubectl get pods -n monitoring -l app.kubernetes.io/name=promtail
```

Example dashboard: [loki-dashboard.json](../dashboards/loki-dashboard.json)


## 5. Prometheus Queries

### CPU Usage (%)
```
sum (rate (container_cpu_usage_seconds_total{namespace="default"}[1m])) / sum (machine_cpu_cores) * 100
```

###  Memory Usage (by Pod)
```
sum (container_memory_usage_bytes{namespace="default"}) by (pod)
```

### Network Receive (bytes per 5m)
```
sum(rate(container_network_receive_bytes_total{namespace="default"}[5m])) by (pod)
```

### Network Transmit (bytes per 5m)
```
sum(rate(container_network_transmit_bytes_total{namespace="default"}[5m])) by (pod)
```

## Cloud / Firewall Note

 - Expose Port `9090` (Prometheus) and `31000` (Grafana) in Security Group/Firewall Rules if using Cloud and want to access externally. Loki is a ClusterIP service and should normally remain internal.

