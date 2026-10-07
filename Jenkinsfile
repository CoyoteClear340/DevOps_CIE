pipeline {
  agent any
  triggers {
    // Enables "GitHub hook trigger for GITScm polling" in the job config.
    // A push to GitHub then auto-starts the build via http://<jenkins-host>/github-webhook/
    // If Jenkins is only on localhost (not reachable from github.com), use Poll SCM
    // as fallback or expose Jenkins with ngrok. See README "Jenkins CI/CD".
    githubPush()
  }
  options {
    timestamps()
    timeout(time: 30, unit: 'MINUTES')
    buildDiscarder(logRotator(numToKeepStr: '10'))
    disableConcurrentBuilds()
  }
  parameters {
    string(name: 'DOCKER_IMAGE', defaultValue: 'arnavk11/devops-cie', description: 'Docker Hub repository, for example username/devops-monitor-dashboard')
  }
  environment {
    IMAGE_TAG = "build-${BUILD_NUMBER}"
    K8S_NAMESPACE = "devops-demo"
    DOCKER_CREDENTIALS_ID = "dockerhub-credentials"
  }
  stages {
    stage('Checkout') { steps { checkout scm } }
    stage('Validate') {
      steps {
        sh 'docker version >/dev/null && kubectl version --client >/dev/null && echo "tools OK"'
        sh 'test -f app/index.html && test -f app/style.css && test -f app/script.js'
        sh 'test -f docker/Dockerfile && test -f nginx/nginx.conf'
        sh 'kubectl apply --dry-run=client -f k8s/namespace.yaml -f k8s/deployment.yaml -f k8s/service.yaml -f k8s/exporter-service.yaml >/dev/null'
        sh 'kubectl apply --dry-run=client -f k8s/prometheus-config.yaml -f k8s/prometheus-deployment.yaml -f k8s/prometheus-service.yaml >/dev/null'
        sh 'kubectl apply --dry-run=client -f k8s/grafana-datasource.yaml -f k8s/grafana-dashboard-provider.yaml -f k8s/grafana-dashboard.yaml -f k8s/grafana-deployment.yaml -f k8s/grafana-service.yaml >/dev/null'
      }
    }
    stage('Build Docker Image') {
      steps { sh 'docker build -f docker/Dockerfile -t "$DOCKER_IMAGE:$IMAGE_TAG" -t "$DOCKER_IMAGE:latest" .' }
    }
    stage('Test Docker Container') {
      steps {
        sh '''
          set -e
          docker rm -f devops-dashboard-test >/dev/null 2>&1 || true
          docker run -d --name devops-dashboard-test -p 18080:8080 "$DOCKER_IMAGE:$IMAGE_TAG"
          OK=0
          for i in $(seq 1 20); do
            if curl --fail --silent http://127.0.0.1:18080/ | grep -q "DevOps Monitor Dashboard"; then OK=1; break; fi
            sleep 2
          done
          docker rm -f devops-dashboard-test >/dev/null 2>&1 || true
          test "$OK" = "1"
        '''
      }
    }
    stage('Push Docker Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
          sh 'echo "$DOCKER_PASSWORD" | docker login --username "$DOCKER_USERNAME" --password-stdin'
          sh 'docker push "$DOCKER_IMAGE:$IMAGE_TAG" && docker push "$DOCKER_IMAGE:latest"'
        }
      }
    }
    stage('Deploy to Kubernetes') {
      steps {
        sh 'kubectl apply -f k8s/namespace.yaml -f k8s/deployment.yaml -f k8s/service.yaml -f k8s/exporter-service.yaml'
        sh 'kubectl apply -f k8s/prometheus-config.yaml -f k8s/prometheus-deployment.yaml -f k8s/prometheus-service.yaml'
        sh 'kubectl apply -f k8s/grafana-datasource.yaml -f k8s/grafana-dashboard-provider.yaml -f k8s/grafana-dashboard.yaml -f k8s/grafana-deployment.yaml -f k8s/grafana-service.yaml'
        sh 'kubectl -n "$K8S_NAMESPACE" set image deployment/devops-dashboard nginx="$DOCKER_IMAGE:$IMAGE_TAG"'
      }
    }
    stage('Verify Kubernetes Deployment') {
      steps {
        sh 'kubectl -n "$K8S_NAMESPACE" rollout status deployment/devops-dashboard --timeout=180s'
        sh 'test "$(kubectl -n "$K8S_NAMESPACE" get pods -l app=devops-dashboard --field-selector=status.phase=Running --no-headers | wc -l)" -ge 2'
        sh 'kubectl -n "$K8S_NAMESPACE" get pods,svc'
      }
    }
    stage('Verify Application') {
      steps {
        sh '''
          set -e
          kubectl -n "$K8S_NAMESPACE" port-forward service/devops-dashboard 18081:8080 >/tmp/devops-port-forward.log 2>&1 & PF=$!
          sleep 5
          curl --fail --silent http://127.0.0.1:18081/ | grep -q "DevOps Monitor Dashboard"
          RC=$?
          kill $PF 2>/dev/null || true
          wait $PF 2>/dev/null || true
          exit $RC
        '''
      }
    }
  }
  post {
    always {
      sh 'docker rm -f devops-dashboard-test >/dev/null 2>&1 || true'
      sh 'docker logout >/dev/null 2>&1 || true'
    }
    success { echo 'DevOps Monitor Dashboard deployed successfully.' }
    failure { echo 'Pipeline failed. Check: Docker Hub credentials (dockerhub-credentials), DOCKER_IMAGE param, kubectl context, and stage logs above.' }
  }
}
