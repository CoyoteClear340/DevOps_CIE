# Architecture

```text
GitHub/Git
    │ webhook or manual build
    ▼
Jenkins ── docker build/test/push ──► Docker Hub
    │ kubectl apply + set image
    ▼
Kubernetes namespace: devops-demo
    ├── devops-dashboard Deployment (2 Nginx + exporter sidecars)
    ├── NodePort 30080 ── browser
    ├── exporter Service 9113 ──► Prometheus
    ├── Prometheus NodePort 30090
    └── Grafana NodePort 30300 ──► Prometheus
```

Nginx serves the static site on port 8080 and exposes its local `stub_status` endpoint. The exporter reads that endpoint and converts real Nginx counters into Prometheus metrics. Prometheus scrapes the exporter Service; Grafana queries Prometheus.
