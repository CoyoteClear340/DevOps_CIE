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
            defaultValue: 'whalewarrior456/devops-monitor-dashboard',
            description: 'Docker Hub image repository: username/repository'
        )
    }

environment {
    DOCKER_IMAGE = "${params.DOCKER_IMAGE}"
    IMAGE_TAG = "build-${BUILD_NUMBER}"
    K8S_NAMESPACE = 'devops-demo'
    DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
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

                    Write-Host "Using Docker CLI from PATH:"
                    (Get-Command docker.exe).Source
                    docker.exe version

                    docker.exe build `
                        -f docker/Dockerfile `
                        -t "$env:DOCKER_IMAGE`:$env:IMAGE_TAG" `
                        .

                    if ($LASTEXITCODE -ne 0) {
                        throw "Docker image build failed."
                    }
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

                        $env:DOCKER_PASSWORD | docker.exe login `
                            --username "$env:DOCKER_USERNAME" `
                            --password-stdin

                        if ($LASTEXITCODE -ne 0) {
                            throw "Docker Hub login failed."
                        }

                        docker.exe push "$env:DOCKER_IMAGE`:$env:IMAGE_TAG"

                        if ($LASTEXITCODE -ne 0) {
                            throw "Docker image push failed."
                        }
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    if ([string]::IsNullOrWhiteSpace($env:KUBECONFIG)) {
                        throw "KUBECONFIG is not configured for the Jenkins agent."
                    }

                    if (!(Test-Path $env:KUBECONFIG)) {
                        throw "KUBECONFIG file was not found: $env:KUBECONFIG"
                    }

                    Write-Host "Using Kubernetes configuration: $env:KUBECONFIG"
                    kubectl.exe config current-context

                    kubectl.exe apply `
                        -f k8s/namespace.yaml `
                        -f k8s/deployment.yaml `
                        -f k8s/service.yaml `
                        -f k8s/exporter-service.yaml

                    if ($LASTEXITCODE -ne 0) {
                        throw "Kubernetes resource deployment failed."
                    }

                    kubectl.exe -n "$env:K8S_NAMESPACE" set image `
                        deployment/devops-dashboard `
                        nginx="$env:DOCKER_IMAGE`:$env:IMAGE_TAG"

                    if ($LASTEXITCODE -ne 0) {
                        throw "Kubernetes image update failed."
                    }
                '''
            }
        }

        stage('Verify Deployment and Application') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    kubectl.exe -n "$env:K8S_NAMESPACE" rollout status `
                        deployment/devops-dashboard `
                        --timeout=180s

                    if ($LASTEXITCODE -ne 0) {
                        throw "Kubernetes rollout failed."
                    }

                    kubectl.exe -n "$env:K8S_NAMESPACE" get pods,svc

                    $portForward = Start-Process `
                        -FilePath "kubectl.exe" `
                        -ArgumentList "-n $env:K8S_NAMESPACE port-forward service/devops-dashboard 18081:8080" `
                        -PassThru `
                        -NoNewWindow

                    try {
                        Start-Sleep -Seconds 5

                        $response = Invoke-WebRequest `
                            -Uri "http://127.0.0.1:18081/" `
                            -UseBasicParsing `
                            -TimeoutSec 10

                        if ($response.StatusCode -ne 200) {
                            throw "Application returned HTTP status $($response.StatusCode)."
                        }

                        if ($response.Content -notmatch "DevOps Monitor Dashboard") {
                            throw "Application response did not contain the expected dashboard."
                        }

                        Write-Host "End-to-end application verification passed."
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
                docker.exe logout 2>$null
            '''
        }
        success {
            echo 'DevOps Monitor Dashboard deployed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check Docker Hub credentials, DOCKER_IMAGE, KUBECONFIG, and stage logs.'
        }
    }
}
