# Deliberate Troubleshooting Scenarios

## 1. Wrong image: `ImagePullBackOff`

Change the image temporarily: `kubectl -n devops-demo set image deployment/devops-dashboard nginx=does-not-exist/dashboard:bad`. Check `kubectl -n devops-demo get pods` and `kubectl -n devops-demo describe pod <pod>`. `ImagePullBackOff` and an Events message about pulling the image mean Kubernetes cannot find or authenticate to the requested tag. Fix with `kubectl -n devops-demo set image deployment/devops-dashboard nginx=<your-user>/devops-monitor-dashboard:latest`, then verify with `kubectl -n devops-demo rollout status deployment/devops-dashboard`.

## 2. Wrong Service port: inaccessible application

Run `kubectl -n devops-demo patch service devops-dashboard -p '{"spec":{"ports":[{"name":"http","port":8080,"targetPort":9999}]}}'`. `kubectl -n devops-demo get svc devops-dashboard`, `kubectl -n devops-demo describe svc devops-dashboard`, and `kubectl -n devops-demo get endpoints devops-dashboard` show that endpoints exist but traffic is sent to a port no container listens on. Fix with `kubectl -n devops-demo patch service devops-dashboard -p '{"spec":{"ports":[{"name":"http","port":8080,"targetPort":"http","nodePort":30080}]}}'`, then curl the service again.

## 3. Prometheus target down

Temporarily edit `k8s/prometheus-config.yaml` to use port `9114`, apply it, and restart Prometheus: `kubectl apply -f k8s/prometheus-config.yaml && kubectl -n devops-demo rollout restart deployment/prometheus`. Check `kubectl -n devops-demo logs deployment/prometheus` and the Prometheus Targets page. DNS/service errors or connection refused identify the bad scrape target. Restore port `9113`, apply, restart, and confirm `up{job="nginx-exporter"} == 1`.
