
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'hsri123/online-bookstore'
        DOCKER_BIN = 'C:\\Users\\srish\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin'
    }

    stages {

        
stage('Checkout') {
    steps {
        deleteDir()
        git branch: 'main',
            url: 'https://github.com/SRISHTI-123-BYTE/Q2_Online_Bookstore.git'
    }
}


        stage('Check Docker') {
            steps {
                bat '''
                    docker --version
                    docker info
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '''
                    docker build --no-cache -t %IMAGE_NAME%:%BUILD_NUMBER% .
                    docker tag %IMAGE_NAME%:%BUILD_NUMBER% %IMAGE_NAME%:latest
                '''
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
                        docker push %IMAGE_NAME%:latest
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kubeconfig',
                        variable: 'KUBECONFIG'
                    )
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
                    file(
                        credentialsId: 'kubeconfig',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    withEnv(["PATH=${DOCKER_BIN};${env.PATH}"]) {
                        bat '''
                            kubectl.exe get deployments
                            kubectl.exe get pods
                            kubectl.exe get services
                            kubectl.exe get deployment bookstore-deployment -o jsonpath="{.spec.template.spec.containers[0].image}"
                            echo.
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Docker image built, pushed, and deployed to Kubernetes.'
        }

        failure {
            echo 'FAILED: Check the first red error in the Jenkins console output.'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}
