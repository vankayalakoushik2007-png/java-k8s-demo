pipeline {
    agent any

    environment {
        IMAGE = '24MIS0341/java-k8s-demo'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Test') {
            steps {
                bat 'docker build -t %IMAGE%:%BUILD_NUMBER% .'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    powershell '''
                        $env:DOCKER_PASS | docker login `
                            --username $env:DOCKER_USER --password-stdin
                        if ($LASTEXITCODE -ne 0) {
                            throw "Docker login failed"
                        }
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                bat 'docker push %IMAGE%:%BUILD_NUMBER%'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat 'kubectl apply -f k8s/service.yaml'
                bat 'kubectl apply -f k8s/deployment.yaml'
                bat 'kubectl set image deployment/java-web-deployment java-web=%IMAGE%:%BUILD_NUMBER%'
                bat 'kubectl rollout status deployment/java-web-deployment --timeout=180s'
            }
        }
    }
}