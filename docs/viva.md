# Viva Questions and Answers

## Git
- **Why Git?** It records changes, supports collaboration and makes rollback possible.
- **What is a branch?** An independent line of development.
- **Why branches?** Features can be tested without destabilizing `main`.
- **Push vs pull?** Push sends local commits to a remote; pull downloads and merges remote commits.
- **What is merge?** Combining one branch's commits into another.

## Jenkins
- **What is CI/CD?** Continuous Integration validates changes; Continuous Delivery/Deployment releases them automatically.
- **What is Jenkins?** An automation server that runs the pipeline from the Jenkinsfile.
- **What triggers it?** A GitHub webhook, polling, or a manual Build Now action.
- **Why credentials?** Secrets stay in Jenkins' encrypted credential store rather than source code.
- **Stages?** Validate, build/test an image, push it, deploy Kubernetes resources, and verify health.

## Docker
- **Image/container?** An image is an immutable package; a container is a running instance.
- **FROM/COPY/RUN/CMD?** Base image, copy files, build-time command, and default runtime command.
- **Why Docker?** The same image runs consistently on a laptop, Jenkins and Kubernetes.

## Kubernetes
- **Pod?** The smallest schedulable unit, containing one or more containers.
- **Deployment?** Maintains the desired replica count and performs rolling updates.
- **Service?** A stable virtual IP/DNS name that routes to matching Pods.
- **Why replicas/scaling?** Availability and capacity; Kubernetes recreates failed Pods.
- **Deployment vs Service?** Deployment manages workloads; Service provides network access.

## Prometheus and Grafana
- **Prometheus?** A time-series database that periodically scrapes metrics endpoints.
- **Target/exporter?** A target is an endpoint to scrape; an exporter translates another system's data into Prometheus format.
- **Metric/PromQL?** A named numeric time series; PromQL queries and calculates those series.
- **Why Grafana?** It turns Prometheus queries into dashboards and visual panels.

## Integration
Git stores the change, Jenkins builds and tests the image, Docker packages it, Kubernetes runs replicas, the exporter exposes real Nginx counters, Prometheus stores them, and Grafana visualizes PromQL. Debug from the outside inward: Service, Endpoints, Pods/events, logs, then the monitoring target.
