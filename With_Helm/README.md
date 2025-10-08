## 1. Install Docker, Kind, Kubectl
- Follow this guide to install Docker: [Link](https://github.com/Abhishek-2502/Java_Jenkins_Docker_Setup_Cloud)
- Follow this guide to install Kind and Kubectl: [Link](https://github.com/Abhishek-2502/K8s_Basics/tree/main/KIND_Cluster)

## 2. Install HELM 

### Linux
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
```
```
chmod 700 get_helm.sh
```
```
./get_helm.sh
```

### Windows (via Chocolatey)

```
choco install kubernetes-helm -y
```
```
helm version
```

## 3. Install Kube Prometheus Stack 

### Add Helm Repositories
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```
```
helm repo add stable https://charts.helm.sh/stable
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
helm install kind-prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --set prometheus.service.nodePort=30000 --set prometheus.service.type=NodePort --set grafana.service.nodePort=31000 --set grafana.service.type=NodePort --set alertmanager.service.nodePort=32000 --set alertmanager.service.type=NodePort --set prometheus-node-exporter.service.nodePort=32001 --set prometheus-node-exporter.service.type=NodePort
```

### Verify Installation
```
kubectl get svc -n monitoring
```
```
kubectl get namespace
```
```
helm list
```
### Uninstall (if needed)
```
helm uninstall kind-prometheus -n monitoring
```

---

## 4. Access Prometheus & Grafana

### Linux
```
kubectl port-forward svc/kind-prometheus-kube-prome-prometheus -n monitoring 9090:9090 --address=0.0.0.0 &
```
```
kubectl port-forward svc/kind-prometheus-grafana -n monitoring 31000:80 --address=0.0.0.0 &
```

### Windows
```
kubectl port-forward svc/kind-prometheus-kube-prome-prometheus -n monitoring 9090:9090 
```
```
kubectl port-forward svc/kind-prometheus-grafana -n monitoring 31000:80 
```

#### Access in Browser:

 - Prometheus: http://localhost:9090

 - Grafana: http://localhost:31000

 - Prometheus Metrics: http://localhost:9090/metrics

Grafana Credentials
 - `username:` admin
 - `password:` prom-operator


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

 - Expose Port `9090` (Prometheus) and `31000` (Grafana) in Security Group/Firewall Rules if using Cloud and want to access externally.

