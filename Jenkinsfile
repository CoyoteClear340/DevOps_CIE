pipeline {
    agent any

    options {
        timestamps()
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
    }

    environment {
        KUBECONFIG = 'C:\\Users\\AYUSH\\.kube\\config'
        IMAGE_NAME = 'task-manager'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Environment Check') {
            steps {
                bat 'docker --version'
                bat 'kubectl version --client'
                bat 'kubectl config current-context'
                bat 'kubectl get nodes'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npm install'
            }
        }

        stage('Build') {
            steps {
                bat 'npm run build'
            }
        }

        stage('Automated Testing') {
            steps {
                bat 'npm test'
            }
        }

        stage('Docker Build') {
            steps {
                bat 'docker build -t %IMAGE_NAME%:%BUILD_NUMBER% .'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat 'kubectl apply -f deployment.yaml'
                bat 'kubectl apply -f service.yaml'

                bat 'kubectl set image deployment/task-manager-deployment task-manager=%IMAGE_NAME%:%BUILD_NUMBER%'

                bat 'kubectl rollout status deployment/task-manager-deployment --timeout=120s'
            }
        }

        stage('Verify Kubernetes') {
            steps {
                bat 'kubectl get deployment task-manager-deployment -o wide'
                bat 'kubectl get pods'
                bat 'kubectl get service task-manager-service'
            }
        }
    }

    post {
        success {
            echo 'Student Task Manager deployed successfully to Kubernetes.'
        }

        failure {
            echo 'Pipeline failed. Check the stage logs.'
        }
    }
}
