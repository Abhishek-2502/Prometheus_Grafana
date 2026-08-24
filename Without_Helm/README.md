### Apply Prometheus config:
```bash
kubectl apply -f prometheus-config.yaml
kubectl apply -f storage.yaml
```

### Deploy Loki:
```bash
kubectl apply -f loki-config.yaml
kubectl apply -f loki-deployment.yaml
kubectl apply -f loki-service.yaml
```

### Deploy Promtail log collector:
```bash
kubectl apply -f promtail-config.yaml
kubectl apply -f promtail-rbac.yaml
kubectl apply -f promtail-daemonset.yaml
```

### Deploy Prometheus:
```bash
kubectl apply -f prometheus-deployment.yaml
kubectl apply -f prometheus-service.yaml
```

### Deploy Grafana:
```bash
kubectl apply -f grafana-deployment.yaml
kubectl apply -f grafana-service.yaml
```

### Verify:
```bash
kubectl get pods
```

### Port-forward to test locally:
```bash
kubectl port-forward svc/prometheus-service 30900:9090
```
```bash
kubectl port-forward svc/loki-service 31000:3100
```
```bash
kubectl port-forward svc/grafana-service 32000:3000
```
```bash
kubectl port-forward daemonset/promtail 9080:9080
```

**NOTE:** Change `prometheus-config.yaml` as per your backend.

**Prometheus:** http://localhost:30900

**Grafana:** http://localhost:32000

**Loki** has no web UI at `/`. Check it with:
```
http://localhost:31000/ready
```
It should return `ready`. Use Grafana to browse logs through the Loki datasource.

**Promtail:** http://localhost:9080

### Add Loki and Prometheus to Grafana:

1. Go to Grafana UI.

2. Login (default: admin/admin)

3. Add Prometheus as a Data Source (or your NodePort URL): 
```
http://prometheus-service:9090
``` 

4. Add Loki as a data source:
```
http://loki-service:3100
```

5. Click Dashboard → Import → Upload JSON

6. Select [loki-dashboard.json](../../dashboards/loki-dashboard.json) and click Import.

7. In the import dialog, map the `DS_LOKI` input to the Loki datasource you created above, then click Import.

The Promtail DaemonSet sends Kubernetes pod logs to Loki. After applying it, wait for the Promtail pod to become `Running`, then refresh the dashboard and select a namespace and pod.

**NOTE:** Loki uses `emptyDir` storage in this example. Logs are lost when its pod is recreated; use a persistent volume for production.

### Cleanup:

Run this command from the `Without_Helm` directory:
```bash
kubectl delete -f . --ignore-not-found
```

Verify that the monitoring resources were removed:
```bash
kubectl get all -l 'app in (grafana,loki,prometheus,promtail)' --ignore-not-found
```

