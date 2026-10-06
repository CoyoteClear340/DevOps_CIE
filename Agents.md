You are an expert DevOps engineer. Build a COMPLETE, WORKING, LOCAL DevOps demonstration project from scratch.

IMPORTANT:

* Do NOT only give me a summary, architecture, explanation, or steps.
* You must actually CREATE all required project files and configurations in the current workspace.
* The final project must be runnable locally.
* Prefer simple technologies because the primary objective is demonstrating DevOps concepts, not application development.
* Do not add unnecessary complexity.
* The application itself can be a simple static HTML page.
* Every DevOps component must be genuinely configured and connected.
* The project must be suitable for a college DevOps viva/demo where I need to demonstrate commands and explain what is happening.

==================================================
PROJECT OBJECTIVE
=================

Create a simple cloud-native DevOps project called:

"DevOps Monitor Dashboard"

The application is a simple static HTML/CSS/JavaScript page served by Nginx.

The main objective is to demonstrate this complete pipeline:

Git
↓
Jenkins CI/CD
↓
Docker
↓
Kubernetes
↓
Prometheus
↓
Grafana

The project must demonstrate:

1. Git version control
2. Git branching and merge workflow
3. Jenkins CI/CD pipeline
4. Docker image creation
5. Docker container execution
6. Kubernetes Deployment
7. Kubernetes Service
8. Kubernetes replicas/scaling
9. Kubernetes troubleshooting
10. Prometheus monitoring
11. Application metrics
12. Grafana dashboard
13. End-to-end integration
14. Deliberate failure/troubleshooting scenario

==================================================
TECHNOLOGY CONSTRAINTS
======================

Use:

Frontend:

* HTML
* CSS
* Vanilla JavaScript

Web server:

* Nginx

Container:

* Docker

CI/CD:

* Jenkins

Container orchestration:

* Kubernetes
* The project should work with Docker Desktop Kubernetes OR Minikube.
* Prefer Kubernetes manifests that work with either.

Monitoring:

* Prometheus
* Grafana

Version control:

* Git
* GitHub-compatible repository structure

Do NOT use:

* React
* Node.js application server
* PostgreSQL
* Redis
* cloud services
* paid services
* Terraform
* Ansible
* complicated backend
* unnecessary frameworks

The purpose is DevOps demonstration, not application development.

==================================================
APPLICATION
===========

Create a visually clean but extremely simple webpage.

Title:

"DevOps Monitor Dashboard"

Show:

* Application name
* Environment: Kubernetes
* Version
* Deployment status
* Container status
* Number of replicas
* Monitoring status
* A short description of the Git → Jenkins → Docker → Kubernetes → Prometheus → Grafana pipeline

The page should look professional enough for a college demonstration.

Create:

index.html
style.css
script.js

Use Nginx to serve the page.

The page itself does not need a backend.

==================================================
IMPORTANT MONITORING REQUIREMENT
================================

Because the application is static, configure monitoring in a way that is simple but actually demonstrable.

Use an Nginx Prometheus exporter OR another simple Prometheus-compatible metrics endpoint.

The final Kubernetes setup must allow Prometheus to scrape useful metrics.

At minimum, expose useful metrics related to:

* HTTP requests
* HTTP response status
* request rate
* active connections
* response/request information where available

If using nginx-prometheus-exporter, configure it correctly and document exactly how it works.

Do NOT fake metrics.

Prometheus must actually scrape a real target.

==================================================
PROJECT STRUCTURE
=================

Create a clean structure similar to:

devops-monitor-dashboard/
│
├── app/
│   ├── index.html
│   ├── style.css
│   └── script.js
│
├── nginx/
│   └── nginx.conf
│
├── docker/
│   └── Dockerfile
│
├── k8s/
│   ├── namespace.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── exporter-deployment.yaml
│   ├── exporter-service.yaml
│   ├── prometheus-config.yaml
│   ├── prometheus-deployment.yaml
│   ├── prometheus-service.yaml
│   ├── grafana-deployment.yaml
│   ├── grafana-service.yaml
│   └── ...
│
├── monitoring/
│   ├── prometheus.yml
│   └── grafana-dashboard.json
│
├── Jenkinsfile
├── .gitignore
├── README.md
└── docs/
├── architecture.md
├── commands.md
├── troubleshooting.md
└── viva.md

You may modify the structure if you have a better simple structure, but keep it understandable for a student viva.

==================================================
DOCKER
======

Create a proper Dockerfile.

Use an appropriate lightweight Nginx base image.

The Dockerfile must clearly demonstrate:

FROM
COPY
RUN where genuinely required
CMD or ENTRYPOINT where appropriate

Do NOT add unnecessary commands just to satisfy the rubric.

The image should:

1. Build successfully.
2. Be tagged properly.
3. Run successfully.
4. Expose the correct HTTP port.
5. Serve the static webpage.

Use a configurable image name.

Example:

DOCKER_IMAGE=<dockerhub-username>/devops-monitor-dashboard

Do NOT hardcode my Docker Hub username.

Use a Jenkins parameter/environment variable for the image repository.

==================================================
JENKINS CI/CD
=============

Create a complete Jenkinsfile.

The Jenkins pipeline must contain real stages.

Use approximately:

1. Checkout
2. Validate
3. Build Docker Image
4. Test Docker Container
5. Push Docker Image
6. Deploy to Kubernetes
7. Verify Kubernetes Deployment
8. Verify Application

The pipeline must be realistic and executable.

Important:

Use Jenkins credentials correctly.

Create a credential requirement such as:

dockerhub-credentials

Use Jenkins credentials binding rather than putting a Docker Hub username/password directly in the Jenkinsfile.

The Jenkinsfile should NOT contain:

docker username
docker password
GitHub token
secret values

Use Jenkins credentials IDs.

Make Docker Hub repository configurable using:

DOCKER_IMAGE

Example:

yourusername/devops-monitor-dashboard

Do not assume the repository name.

==================================================
JENKINS PIPELINE BEHAVIOR
=========================

The pipeline should:

1. Checkout source code.
2. Validate that required files exist.
3. Build the Docker image.
4. Start a temporary container.
5. Verify the webpage responds successfully.
6. Stop/remove the temporary container.
7. Login to Docker Hub using Jenkins credentials.
8. Push the image.
9. Deploy/update Kubernetes resources.
10. Wait for rollout.
11. Verify pods are running.
12. Verify the service.
13. Print useful deployment information.

Make the pipeline fail if:

* Docker build fails.
* Container does not start.
* HTTP check fails.
* Docker push fails.
* Kubernetes deployment fails.
* Kubernetes rollout fails.

Use shell commands appropriate for a Linux Jenkins agent.

If a Windows Jenkins agent is possible, document the required change separately, but keep the main Jenkinsfile Linux-compatible.

==================================================
DOCKER IMAGE TAGGING
====================

Use meaningful tags.

For example:

build-${BUILD_NUMBER}

and optionally:

latest

The Kubernetes deployment should be updated to the new image tag.

Do not blindly deploy "latest" if a build-specific tag can be used.

==================================================
KUBERNETES
==========

Create complete Kubernetes YAML manifests.

Use a namespace:

devops-demo

Create:

Deployment
Service

The application Deployment should initially use:

replicas: 2

This is important because replica scaling needs to be demonstrated.

The Deployment should:

* use the Docker image
* expose the application port
* define appropriate labels
* define readiness probe
* define liveness probe
* define resource requests/limits if reasonable
* use a rolling update strategy

The Service should expose the application.

For local development, choose the simplest reliable exposure method:

NodePort

or another method that works easily with Minikube/Docker Desktop.

Document the exact command to access it.

==================================================
KUBERNETES MONITORING
=====================

Deploy/configure Prometheus inside Kubernetes.

Prometheus must be able to scrape the application/exporter.

Configure:

prometheus.yml

with an actual scrape job.

Use Kubernetes-compatible service discovery where practical, or use a simple static scrape target if that is significantly easier and more reliable for this student project.

Do not over-engineer service discovery.

The Prometheus setup must be understandable enough to explain during viva.

==================================================
GRAFANA
=======

Deploy Grafana in Kubernetes.

Configure it to use Prometheus as its data source.

Create a dashboard automatically if possible.

Create useful panels such as:

1. HTTP request rate
2. Total HTTP requests
3. Active connections
4. HTTP response/status information
5. Pod availability if practical

Use real PromQL queries.

Create:

monitoring/grafana-dashboard.json

if dashboard provisioning is practical.

Otherwise create the dashboard setup instructions in README.

Prefer automatic provisioning so the demo is reproducible.

==================================================
PROMETHEUS METRICS
==================

Use real Prometheus metrics.

The README must explain:

What is Prometheus?

What is a scrape target?

What is an exporter?

What metric is being collected?

What is the PromQL query?

How does traffic generate a metric?

For example, if the exporter exposes nginx metrics, demonstrate queries such as request rate using an appropriate metric actually available from the exporter.

DO NOT invent metric names.

Verify the actual metric names and use those exact names in Grafana.

==================================================
GENERATING TRAFFIC
==================

Provide simple commands to generate traffic.

For example:

curl

or a shell loop.

The README must show how to generate traffic against the Kubernetes service so that Prometheus/Grafana graphs visibly change.

Example concept:

for i in {1..100}; do curl http://...; done

Adapt the command to the actual Kubernetes setup.

==================================================
GIT WORKFLOW
============

The project must demonstrate Git properly.

README must include commands for:

git init
git add
git commit
git branch
git checkout/switch
git merge
git remote
git push
git pull
git clone

Create a recommended workflow:

main
develop
feature/devops-dashboard

Explain:

feature branch
→ commit
→ push
→ merge/pull request
→ main
→ Jenkins pipeline

Do not just describe Git theoretically.

==================================================
TROUBLESHOOTING DEMONSTRATION
=============================

This is VERY IMPORTANT.

Create a docs/troubleshooting.md file containing at least 3 realistic failures that can be deliberately introduced during the viva.

Examples:

Failure 1:
Incorrect Docker image name.

Symptoms:
ImagePullBackOff

Commands:

kubectl get pods
kubectl describe pod <pod>

Diagnosis:
Wrong image/repository/tag.

Fix:
Correct image.

Failure 2:
Wrong Kubernetes Service port.

Symptoms:
Application inaccessible.

Commands:

kubectl get svc
kubectl describe svc
kubectl get endpoints

Diagnosis:
Port/targetPort mismatch.

Fix:
Correct Service configuration.

Failure 3:
Prometheus target unavailable.

Symptoms:
Target DOWN.

Commands:
kubectl get pods
kubectl get svc
kubectl logs
Prometheus Targets page

Diagnosis:
Incorrect scrape endpoint/service/configuration.

Fix:
Correct Prometheus configuration.

For each failure explain:

1. Symptom
2. Command used
3. What output means
4. Root cause
5. Fix
6. Verification

==================================================
VIVA DOCUMENTATION
==================

Create:

docs/viva.md

Include simple questions and answers for:

Git:

* Why Git?
* What is a branch?
* Why use branches?
* Difference between pull and push?
* What is merge?

Jenkins:

* What is CI/CD?
* What is Jenkins?
* What triggers the pipeline?
* What happens in each stage?
* Why use Jenkins credentials?

Docker:

* What is Docker?
* What is an image?
* What is a container?
* Explain FROM, COPY, RUN, CMD.
* Image vs container.
* Why Docker?

Kubernetes:

* What is Kubernetes?
* What is a Pod?
* What is a Deployment?
* What is a Service?
* Why replicas?
* What is scaling?
* What happens if a Pod dies?
* Difference between Deployment and Service.

Prometheus:

* What is Prometheus?
* What is scraping?
* What is a target?
* What is an exporter?
* What is a metric?
* What is PromQL?

Grafana:

* What is Grafana?
* Why Grafana if Prometheus already exists?
* What is a dashboard?
* What is a panel?
* How does Grafana get data?

Integration:

* Explain Git → Jenkins → Docker → Kubernetes → Prometheus → Grafana.
* What output does each stage provide to the next?
* Where can the pipeline fail?
* How would you debug a failed deployment?

Keep answers simple enough for a beginner to memorize and understand.

==================================================
README
======

Create a COMPLETE README.md.

It must include:

1. Project overview
2. Architecture diagram using ASCII/Markdown
3. Technologies
4. Folder structure
5. Prerequisites
6. Installation
7. Git setup
8. Docker setup
9. Docker build/run commands
10. Jenkins setup
11. Jenkins credentials setup
12. Jenkins pipeline setup
13. Kubernetes setup
14. Kubernetes deployment commands
15. Service access
16. Prometheus setup
17. Grafana setup
18. Dashboard access
19. Generating traffic
20. Verification commands
21. Troubleshooting
22. Git workflow
23. End-to-end demo procedure
24. Viva questions

The README must contain exact commands.

Do not write vague statements such as:

"Configure Jenkins."

Instead write exactly what needs to be clicked/configured and what values to enter.

==================================================
END-TO-END DEMO
===============

Create a section called:

"8-10 Minute CIE Demo"

The demo should follow this sequence:

1. Show Git repository.
2. Show branch.
3. Make/show a small change.
4. Commit and push.
5. Show Jenkins automatically triggered.
6. Show pipeline stages.
7. Show Docker image.
8. Show Kubernetes Pods.
9. Show Kubernetes Service.
10. Show two replicas.
11. Access webpage.
12. Open Prometheus.
13. Show target UP.
14. Generate traffic.
15. Show metrics.
16. Open Grafana.
17. Show dashboard.
18. Deliberately introduce one small failure.
19. Diagnose using kubectl.
20. Fix it.
21. Verify system works again.

Keep this demo practical and achievable locally.

==================================================
VERIFICATION
============

After creating all files, actually inspect the project and verify consistency.

Check:

* Dockerfile paths
* COPY paths
* nginx configuration
* Kubernetes image names
* ports
* Service targetPort
* Prometheus scrape target
* exporter endpoint
* Grafana datasource
* Grafana PromQL queries
* Jenkins credentials ID
* Jenkins image variable
* Kubernetes namespace
* Deployment names
* Service names

Do not leave broken references.

==================================================
LOCAL ENVIRONMENT
=================

Assume:

Windows host
+
WSL2 Ubuntu

The user may use:

Docker Desktop
Minikube
kubectl
Jenkins

Prefer commands that work inside WSL2.

Document prerequisite installation where required.

Do NOT require a cloud account.

==================================================
SECURITY
========

Never hardcode:

* Docker Hub password
* Docker Hub access token
* GitHub token
* Jenkins secrets

Use environment variables or Jenkins credentials where required.

Add appropriate secret patterns to .gitignore.

==================================================
QUALITY REQUIREMENTS
====================

The project must be:

* Simple
* Reproducible
* Fully connected
* Easy to explain
* Easy to troubleshoot
* Local
* Free
* Suitable for a college DevOps demonstration

Avoid unnecessary production complexity.

The goal is not to build a sophisticated application.

The goal is to demonstrate:

Git
→ CI/CD
→ Docker
→ Kubernetes
→ Monitoring
→ Visualization

==================================================
FINAL AGENT TASK
================

After creating everything:

1. List every file you created.
2. Show the final folder structure.
3. Check for broken references.
4. Explain any assumptions.
5. Tell me exactly which commands I need to run first.
6. Tell me which Jenkins credentials I need to create.
7. Tell me what Docker Hub repository I need.
8. Tell me how to start Kubernetes.
9. Tell me how to deploy manually once.
10. Tell me how Jenkins will deploy automatically.
11. Tell me how to open the application.
12. Tell me how to open Prometheus.
13. Tell me how to open Grafana.
14. Tell me how to generate traffic.
15. Tell me how to demonstrate the troubleshooting scenario.

MOST IMPORTANT:

Do not stop after generating a plan.

Actually create the complete project files in the current workspace.

If you encounter an error while creating or validating the project, diagnose it and fix the files rather than simply reporting the error.

Do not leave TODO placeholders.

Do not use fake configuration.

Do not claim something is working unless you have actually verified the configuration as far as the available environment allows.

The final result should be a complete DevOps project that can be demonstrated against the CIE rubric.
