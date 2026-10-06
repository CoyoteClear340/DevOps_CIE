# Command Cheat Sheet

```bash
kubectl config current-context
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/deployment.yaml -f k8s/service.yaml -f k8s/exporter-service.yaml
kubectl apply -f k8s/prometheus-config.yaml -f k8s/prometheus-deployment.yaml -f k8s/prometheus-service.yaml
kubectl apply -f k8s/grafana-datasource.yaml -f k8s/grafana-dashboard-provider.yaml -f k8s/grafana-dashboard.yaml -f k8s/grafana-deployment.yaml -f k8s/grafana-service.yaml
kubectl -n devops-demo get pods,svc
kubectl -n devops-demo scale deployment/devops-dashboard --replicas=4
kubectl -n devops-demo rollout status deployment/devops-dashboard
kubectl -n devops-demo port-forward svc/devops-dashboard 8080:8080
kubectl -n devops-demo port-forward svc/prometheus 9090:9090
kubectl -n devops-demo port-forward svc/grafana 3000:3000
```

Generate traffic from WSL:

```bash
for i in {1..100}; do curl -s http://localhost:8080/ >/dev/null; done
```

Use `kubectl port-forward svc/devops-dashboard 8080:8080 -n devops-demo` in another terminal first.
