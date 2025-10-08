# REDIS QUERY
## Show all votes
```
LRANGE votes 0 -1
```

## Total number of votes
```
LLEN votes
```

# DOCKER
## Query inside Redis to check data
```
docker exec -it redis redis-cli
```

# K8S
```
kubectl apply -f .
```

## Windows
```
kubectl port-forward svc/redis 6379:6379 
```
```
kubectl port-forward svc/vote 5000:5000 
```

## Linux (run in background)
```
kubectl port-forward svc/redis 6379:6379 --address=0.0.0.0 &
```
```
kubectl port-forward svc/vote 5000:5000 --address=0.0.0.0 &
```

## Query inside Redis to check data
```
kubectl get pods -l app=redis
```
```
kubectl exec -it <redis-pod-name> -- redis-cli
```

## Cloud / Firewall Note

 - Expose Port `5000` (Python Vote App) in Security Group/Firewall Rules if using Cloud and want to access externally.