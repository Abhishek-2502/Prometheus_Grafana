### Apply Prometheus config:
kubectl apply -f prometheus-config.yaml

### Deploy Prometheus
kubectl apply -f prometheus-deployment.yaml
kubectl apply -f prometheus-service.yaml

### Deploy Grafana:
kubectl apply -f grafana-deployment.yaml
kubectl apply -f grafana-service.yaml

### Port-forward to test locally:
kubectl port-forward svc/prometheus-service 30900:9090
kubectl port-forward svc/grafana-service 32000:3000

```
kubectl apply -f prometheus-config.yaml
kubectl apply -f prometheus-deployment.yaml
kubectl apply -f prometheus-service.yaml
kubectl apply -f grafana-deployment.yaml
kubectl apply -f grafana-service.yaml
```

```
kubectl port-forward svc/prometheus-service 30900:9090
kubectl port-forward svc/grafana-service 32000:3000
```

**NOTE:** Change `prometheus-config.yaml` as per your backend.

**Prometheus:** http://localhost:30900

**Grafana:** http://localhost:32000


1. Go to Grafana UI: 
```
http://localhost:32000
```

2. Login (default: admin/admin)

3. Add Prometheus as a Data Source (or your NodePort URL): 
```
http://prometheus-service:9090
``` 

4. Click Dashboard → Import → Upload JSON

5. Select the JSON file → Import

