
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'hsri123/online-bookstore'
        DOCKER_BIN = 'C:\\Users\\srish\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Source code is stored in GitHub.'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %IMAGE_NAME%:%BUILD_NUMBER% .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    bat '''
                        docker login -u "%DOCKER_USER%" -p "%DOCKER_PASS%"
                        docker push %IMAGE_NAME%:%BUILD_NUMBER%
                        docker tag %IMAGE_NAME%:%BUILD_NUMBER% %IMAGE_NAME%:latest
                        docker push %IMAGE_NAME%:latest
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')
                ]) {
                    withEnv(["PATH=${DOCKER_BIN};${env.PATH}"]) {
                        bat '''
                            kubectl.exe set image deployment/bookstore-deployment bookstore-container=%IMAGE_NAME%:%BUILD_NUMBER%
                            kubectl.exe rollout status deployment/bookstore-deployment --timeout=180s
                        '''
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withCredentials([
                    file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')
                ]) {
                    withEnv(["PATH=${DOCKER_BIN};${env.PATH}"]) {
                        bat '''
                            kubectl.exe get deployments
                            kubectl.exe get pods
                            kubectl.exe get services
                            kubectl.exe get deployment bookstore-deployment -o jsonpath="{.spec.template.spec.containers[0].image}"
                        '''
                    }
                }
            }
        }
    }
}
