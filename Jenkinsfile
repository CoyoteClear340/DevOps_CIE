
pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timestamps()
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
    }

    parameters {
        string(
            name: 'DOCKER_IMAGE',
            defaultValue: 'devops-monitor-dashboard',
            description: 'Local Docker image name'
        )
    }

    environment {
        IMAGE_TAG = "build-${BUILD_NUMBER}"
        K8S_NAMESPACE = 'devops-demo'
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

                    Write-Host "=========================================="
                    Write-Host "Building Docker Image"
                    Write-Host "=========================================="

                    Write-Host "Docker:"
                    (Get-Command docker.exe).Source
                    docker.exe version

                    $IMAGE = "$env:DOCKER_IMAGE`:$env:IMAGE_TAG"

                    Write-Host "Building image: $IMAGE"

                    docker.exe build `
                        -f docker/Dockerfile `
                        -t "$IMAGE" `
                        .

                    if ($LASTEXITCODE -ne 0) {
                        throw "Docker image build failed."
                    }

                    Write-Host ""
                    Write-Host "Docker image created successfully:"
                    docker.exe images "$env:DOCKER_IMAGE"
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    Write-Host "=========================================="
                    Write-Host "Deploying to Kubernetes"
                    Write-Host "=========================================="

                    kubectl.exe config current-context

                    Write-Host "Using namespace: $env:K8S_NAMESPACE"

                    kubectl.exe apply `
                        -f k8s/namespace.yaml `
                        -f k8s/deployment.yaml `
                        -f k8s/service.yaml `
                        -f k8s/exporter-service.yaml

                    if ($LASTEXITCODE -ne 0) {
                        throw "Kubernetes resource deployment failed."
                    }

                    $IMAGE = "$env:DOCKER_IMAGE`:$env:IMAGE_TAG"

                    Write-Host "Updating deployment image to: $IMAGE"

                    kubectl.exe -n "$env:K8S_NAMESPACE" set image `
                        deployment/devops-dashboard `
                        nginx="$IMAGE"

                    if ($LASTEXITCODE -ne 0) {
                        throw "Kubernetes image update failed."
                    }

                    Write-Host "Kubernetes image updated successfully."
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    Write-Host "=========================================="
                    Write-Host "Waiting for Kubernetes rollout"
                    Write-Host "=========================================="

                    kubectl.exe -n "$env:K8S_NAMESPACE" rollout status `
                        deployment/devops-dashboard `
                        --timeout=180s

                    if ($LASTEXITCODE -ne 0) {
                        throw "Kubernetes rollout failed."
                    }

                    Write-Host ""
                    Write-Host "Pods:"
                    kubectl.exe -n "$env:K8S_NAMESPACE" get pods -o wide

                    Write-Host ""
                    Write-Host "Services:"
                    kubectl.exe -n "$env:K8S_NAMESPACE" get svc

                    Write-Host ""
                    Write-Host "Deployment:"
                    kubectl.exe -n "$env:K8S_NAMESPACE" get deployment
                '''
            }
        }

        stage('Test Application') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    Write-Host "=========================================="
                    Write-Host "Testing Application"
                    Write-Host "=========================================="

                    $portForward = Start-Process `
                        -FilePath "kubectl.exe" `
                        -ArgumentList "-n $env:K8S_NAMESPACE port-forward service/devops-dashboard 18081:8080" `
                        -PassThru `
                        -NoNewWindow

                    try {
                        Write-Host "Waiting for port-forward..."
                        Start-Sleep -Seconds 5

                        $url = "http://127.0.0.1:18081/"

                        Write-Host "Testing: $url"

                        $response = Invoke-WebRequest `
                            -Uri $url `
                            -UseBasicParsing `
                            -TimeoutSec 10

                        Write-Host "HTTP Status: $($response.StatusCode)"

                        if ($response.StatusCode -ne 200) {
                            throw "Application returned HTTP status $($response.StatusCode)."
                        }

                        if ($response.Content -notmatch "DevOps Monitor Dashboard") {
                            throw "Application response did not contain the expected dashboard."
                        }

                        Write-Host ""
                        Write-Host "=========================================="
                        Write-Host "APPLICATION TEST PASSED"
                        Write-Host "=========================================="
                        Write-Host "DevOps Monitor Dashboard is running."
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
        success {
            echo 'SUCCESS: Docker image built, deployed to Kubernetes, and application test passed.'
        }

        failure {
            echo 'FAILED: Check Docker build, Kubernetes deployment, rollout, or application test logs.'
        }
    }
}
