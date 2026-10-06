# DevOps Monitor Dashboard

A complete local Git → Jenkins → Docker → Kubernetes → Prometheus → Grafana demonstration using a static Nginx site.

## Architecture

```text
Git -> Jenkins -> Docker image -> Kubernetes (2 replicas)
                                      |              |
                                    Nginx       exporter sidecar
                                      |              |
                                  browser <- Service <- Prometheus <- Grafana
```

The Nginx container listens on `8080`. Its real `stub_status` counters are read by `nginx-prometheus-exporter:1.4.2`; Prometheus scrapes the exporter Service on `9113`. The dashboard uses real exporter metrics: `nginx_http_requests_total`, `nginx_connections_active`, and `up`.

## Technologies and structure

HTML/CSS/JavaScript, Nginx, Docker, Jenkins, Kubernetes, Prometheus and Grafana.

```text
app/                         static site
nginx/nginx.conf             Nginx and stub_status
docker/Dockerfile            image definition
k8s/                         namespace, app, monitoring manifests
monitoring/                  Prometheus and Grafana source configuration
Jenkinsfile                  Linux Jenkins pipeline
docs/                        architecture, commands, troubleshooting, viva
```

## Prerequisites (Windows + WSL2)

Install Docker Desktop with Kubernetes enabled, or Minikube with Docker driver, then install `kubectl`, Git, curl and Jenkins. In WSL verify:

```bash
docker version
kubectl version --client
git --version
curl --version
```

For Minikube:

```bash
minikube start --driver=docker
kubectl config use-context minikube
```

For Docker Desktop, enable **Settings → Kubernetes → Enable Kubernetes**, then `kubectl config use-context docker-desktop`.

## Git setup and workflow

```bash
cd /mnt/e/DevOps_CIE
git init
git add .
git commit -m "Initial DevOps dashboard"
git branch -M main
git switch -c develop
git switch -c feature/devops-dashboard
git add app/index.html && git commit -m "Update dashboard"
git remote add origin https://github.com/<your-user>/<your-repository>.git
git push -u origin feature/devops-dashboard
git switch develop && git merge feature/devops-dashboard
git push origin develop
git switch main && git merge develop && git push origin main
git pull origin main
git clone https://github.com/<your-user>/<your-repository>.git
```

Recommended flow: feature branch → commit/push → pull request into `develop` → merge into `main` → Jenkins build.

## Docker build and run

Choose any Docker Hub repository you own; for example `mydockeruser/devops-monitor-dashboard`. Do not put credentials in this repository.

```bash
export DOCKER_IMAGE=mydockeruser/devops-monitor-dashboard
docker build -f docker/Dockerfile -t "$DOCKER_IMAGE:local" .
docker run --rm -d --name devops-dashboard -p 8080:8080 "$DOCKER_IMAGE:local"
curl http://localhost:8080/
docker rm -f devops-dashboard
```

## Kubernetes: first manual deployment

Build and push a tag, replace `mydockeruser` with your Docker Hub username, then substitute the image placeholder in a temporary rendered file:

```bash
export DOCKER_IMAGE=mydockeruser/devops-monitor-dashboard
docker build -f docker/Dockerfile -t "$DOCKER_IMAGE:local" .
docker push "$DOCKER_IMAGE:local"
kubectl apply -f k8s/namespace.yaml
sed "s|IMAGE_PLACEHOLDER|$DOCKER_IMAGE:local|" k8s/deployment.yaml | kubectl apply -f -
kubectl apply -f k8s/service.yaml -f k8s/exporter-service.yaml
kubectl apply -f k8s/prometheus-config.yaml -f k8s/prometheus-deployment.yaml -f k8s/prometheus-service.yaml
kubectl apply -f k8s/grafana-datasource.yaml -f k8s/grafana-dashboard-provider.yaml -f k8s/grafana-dashboard.yaml -f k8s/grafana-deployment.yaml -f k8s/grafana-service.yaml
kubectl -n devops-demo rollout status deployment/devops-dashboard
kubectl -n devops-demo get pods,svc
```

The namespace must exist before namespaced resources:

```bash
kubectl apply -f k8s/namespace.yaml
```

If using Minikube, `minikube service devops-dashboard -n devops-demo --url` gives the application URL. With Docker Desktop use `http://localhost:30080`; port-forward works everywhere:

```bash
kubectl -n devops-demo port-forward svc/devops-dashboard 8080:8080
```

## Prometheus and Grafana

Prometheus is available at `http://localhost:30090` and Grafana at `http://localhost:30300` with Docker Desktop/NodePort. Port-forward alternatives:

```bash
kubectl -n devops-demo port-forward svc/prometheus 9090:9090
kubectl -n devops-demo port-forward svc/grafana 3000:3000
```

Grafana is anonymously viewable for this local demo; the configured admin password is `demo-admin` and must not be reused outside a local classroom environment. Grafana automatically provisions the Prometheus datasource and dashboard.

Useful real queries:

```promql
rate(nginx_http_requests_total[1m])
sum(nginx_http_requests_total)
nginx_connections_active
up{job="nginx-exporter"}
```

Generate traffic while port-forwarding the application:

```bash
for i in {1..100}; do curl -s http://localhost:8080/ >/dev/null; done
```

Wait 15–30 seconds for a scrape, then refresh Prometheus **Graph** or Grafana.

## Jenkins CI/CD

Create a Jenkins **Pipeline** job. Select **Pipeline script from SCM**, choose Git, enter your repository URL and branch `main`, and set script path to `Jenkinsfile`. Enable a GitHub webhook at `http://<jenkins-host>/github-webhook/` if Jenkins is reachable, otherwise use **Build Now**.

Create **Manage Jenkins → Credentials → System → Global credentials → Add Credentials**:

1. **Username with password**, ID `dockerhub-credentials`, username your Docker Hub username, password a Docker Hub access token (not your account password).
2. Jenkins agent must have Docker and kubectl installed and access to the selected Kubernetes context. If using a kubeconfig credential instead, bind it as a secret file and set `KUBECONFIG` in the pipeline agent configuration; do not commit kubeconfig.

At build time set the string parameter `DOCKER_IMAGE` to your repository, e.g. `mydockeruser/devops-monitor-dashboard`. The pipeline validates files, builds `build-${BUILD_NUMBER}` and `latest`, starts a temporary container and curls it, logs in through the credential binding, pushes both tags, applies Kubernetes resources, sets the build-specific image, waits for rollout, and curls a port-forwarded service.

The Jenkins agent needs permissions for `docker`, `kubectl`, and the active cluster context. On a Windows agent, replace each `sh` with equivalent `bat`/PowerShell commands; the supplied Jenkinsfile intentionally targets a Linux/WSL agent.

## Verification

```bash
kubectl -n devops-demo get deployment,pods,svc
kubectl -n devops-demo get endpoints devops-dashboard devops-dashboard-metrics
kubectl -n devops-demo logs deployment/devops-dashboard -c nginx-exporter
kubectl -n devops-demo describe pod -l app=devops-dashboard
```

## 8-10 Minute CIE Demo

1. Show `git branch` and the feature branch.
2. Make a small HTML change, commit and push it.
3. Show Jenkins stages: Validate, Build, Test, Push, Deploy and Verify.
4. Show the Docker Hub build tag.
5. Run `kubectl -n devops-demo get pods`; point out two replicas.
6. Open the application on port `30080` or with port-forward.
7. Open Prometheus and show `up{job="nginx-exporter"} == 1`.
8. Generate 100 curl requests and show `rate(nginx_http_requests_total[1m])`.
9. Open Grafana and show the provisioned dashboard.
10. Demonstrate the wrong image failure from `docs/troubleshooting.md`, inspect Events with `kubectl describe`, restore the image, and verify rollout.

## Troubleshooting and viva

See [docs/troubleshooting.md](docs/troubleshooting.md), [docs/commands.md](docs/commands.md), [docs/architecture.md](docs/architecture.md), and [docs/viva.md](docs/viva.md).
