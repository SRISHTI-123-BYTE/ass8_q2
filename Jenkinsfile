
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'hsri123/online-bookstore'
        DOCKER_BIN = 'C:\\Users\\srish\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code from GitHub'
                git branch: 'main',
                    url: 'https://github.com/SRISHTI-123-BYTE/ass8_q2.git'
            }
        }

        stage('Check Docker') {
            steps {
                withEnv(["PATH=${DOCKER_BIN};${env.PATH}"]) {
                    bat '''
                        where.exe docker.exe
                        docker.exe --version
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                withEnv(["PATH=${DOCKER_BIN};${env.PATH}"]) {
                    bat '''
                        docker.exe build -t %IMAGE_NAME%:%BUILD_NUMBER% .
                        docker.exe tag %IMAGE_NAME%:%BUILD_NUMBER% %IMAGE_NAME%:latest
                    '''
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withEnv(["PATH=${DOCKER_BIN};${env.PATH}"]) {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'dockerhub-creds',
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )
                    ]) {
                        bat '''
                            docker.exe login -u "%DOCKER_USER%" -p "%DOCKER_PASS%"
                            docker.exe push %IMAGE_NAME%:%BUILD_NUMBER%
                            docker.exe push %IMAGE_NAME%:latest
                            docker.exe logout
                        '''
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withEnv(["PATH=${DOCKER_BIN};${env.PATH}"]) {
                    withCredentials([
                        file(
                            credentialsId: 'kubeconfig',
                            variable: 'KUBECONFIG'
                        )
                    ]) {
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
                withEnv(["PATH=${DOCKER_BIN};${env.PATH}"]) {
                    withCredentials([
                        file(
                            credentialsId: 'kubeconfig',
                            variable: 'KUBECONFIG'
                        )
                    ]) {
                        bat '''
                            kubectl.exe get deployments
                            kubectl.exe get pods
                            kubectl.exe get services
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Docker image pushed and Kubernetes deployment verified.'
        }
        failure {
            echo 'FAILED: Check the first error in the console output.'
        }
    }
}
