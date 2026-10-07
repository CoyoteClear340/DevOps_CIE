
pipeline {
  agent any

  triggers {
    githubPush()
  }

  options {
    timestamps()
    timeout(time: 30, unit: 'MINUTES')
    buildDiscarder(logRotator(numToKeepStr: '10'))
    disableConcurrentBuilds()
  }

  parameters {
    string(
      name: 'DOCKER_IMAGE',
      defaultValue: 'whalewarrior456/devops-monitor-dashboard',
      description: 'Docker Hub repository, for example username/devops-monitor-dashboard'
    )
  }

  environment {
    IMAGE_TAG = "build-${BUILD_NUMBER}"
    K8S_NAMESPACE = "devops-demo"
    DOCKER_CREDENTIALS_ID = "dockerhub-credentials"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Docker Image') {
      steps {
        powershell '''
          $ErrorActionPreference = "Stop"

          Write-Host "Building Docker image..."
          Write-Host "Image: $env:DOCKER_IMAGE"
          Write-Host "Tag: $env:IMAGE_TAG"

          docker build `
            -f docker/Dockerfile `
            -t "$env:DOCKER_IMAGE`:$env:IMAGE_TAG" `
            .

          if ($LASTEXITCODE -ne 0) {
              throw "Docker build failed"
          }

          Write-Host "Docker image built successfully."
        '''
      }
    }

    stage('Push Docker Image') {
      steps {
        withCredentials([
          usernamePassword(
            credentialsId: "${DOCKER_CREDENTIALS_ID}",
            usernameVariable: 'DOCKER_USERNAME',
            passwordVariable: 'DOCKER_PASSWORD'
          )
        ]) {
          powershell '''
            $ErrorActionPreference = "Stop"

            Write-Host "Logging into Docker Hub..."

            $env:DOCKER_PASSWORD | docker login `
              --username "$env:DOCKER_USERNAME" `
              --password-stdin

            if ($LASTEXITCODE -ne 0) {
                throw "Docker Hub login failed"
            }

            Write-Host "Pushing build image..."

            docker push "$env:DOCKER_IMAGE`:$env:IMAGE_TAG"

            if ($LASTEXITCODE -ne 0) {
                throw "Docker image push failed"
            }

            Write-Host "Docker image pushed successfully."
          '''
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        powershell '''
          $ErrorActionPreference = "Stop"

          Write-Host "Deploying application to Kubernetes..."

          kubectl apply `
            -f k8s/namespace.yaml `
            -f k8s/deployment.yaml `
            -f k8s/service.yaml `
            -f k8s/exporter-service.yaml

          Write-Host "Updating deployment image..."

          kubectl -n "$env:K8S_NAMESPACE" set image `
            deployment/devops-dashboard `
            nginx="$env:DOCKER_IMAGE`:$env:IMAGE_TAG"

          if ($LASTEXITCODE -ne 0) {
              throw "Failed to update Kubernetes deployment image"
          }

          Write-Host "Kubernetes deployment updated successfully."
        '''
      }
    }

    stage('Verify Kubernetes Deployment') {
      steps {
        powershell '''
          $ErrorActionPreference = "Stop"

          Write-Host "Waiting for Kubernetes rollout..."

          kubectl -n "$env:K8S_NAMESPACE" rollout status `
            deployment/devops-dashboard `
            --timeout=180s

          if ($LASTEXITCODE -ne 0) {
              throw "Kubernetes rollout failed"
          }

          Write-Host ""
          Write-Host "Checking running pods..."

          $runningPods = kubectl -n "$env:K8S_NAMESPACE" get pods `
            -l app=devops-dashboard `
            --field-selector=status.phase=Running `
            --no-headers

          $podCount = @($runningPods).Count

          Write-Host "Running application pods: $podCount"

          if ($podCount -lt 2) {
              throw "Expected at least 2 running application pods, found $podCount"
          }

          Write-Host ""
          Write-Host "Kubernetes resources:"

          kubectl -n "$env:K8S_NAMESPACE" get pods,svc

          Write-Host ""
          Write-Host "Kubernetes deployment verified successfully."
        '''
      }
    }

    stage('Verify Application') {
      steps {
        powershell '''
          $ErrorActionPreference = "Stop"

          Write-Host "Starting Kubernetes port-forward..."

          $portForward = Start-Process `
            -FilePath "kubectl.exe" `
            -ArgumentList "-n $env:K8S_NAMESPACE port-forward service/devops-dashboard 18081:8080" `
            -PassThru `
            -NoNewWindow

          try {

              Start-Sleep -Seconds 5

              Write-Host "Testing application..."

              $response = Invoke-WebRequest `
                -Uri "http://127.0.0.1:18081/" `
                -UseBasicParsing `
                -TimeoutSec 10

              if ($response.Content -notmatch "DevOps Monitor Dashboard") {
                  throw "Application response did not contain expected dashboard text."
              }

              Write-Host "Application verification successful."

          }
          finally {

              if ($portForward -and -not $portForward.HasExited) {
                  Stop-Process -Id $portForward.Id -Force
              }

          }
        '''
      }
    }
  }

  post {

    always {
      powershell '''
        Write-Host "Cleaning up..."

        docker logout 2>$null

        Write-Host "Cleanup completed."
      '''
    }

    success {
      echo 'DevOps Monitor Dashboard deployed successfully.'
    }

    failure {
      echo 'Pipeline failed. Check Docker Hub credentials, DOCKER_IMAGE parameter, kubectl context, and stage logs above.'
    }
  }
}
