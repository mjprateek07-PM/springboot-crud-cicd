pipeline {
    agent any

    tools {
        maven 'Maven-3.9'  // Must match the name in Jenkins Tools configuration
    }

    environment {
        DOCKER_IMAGE = 'mjprateek07-PM/crud-app'
        DOCKER_TAG = "${BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        KUBECONFIG_CREDENTIAL = 'kubeconfig'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code from GitHub...'
                git branch: 'main',
                    url: 'https://github.com/mjprateek07-PM/springboot-crud-cicd.git'
            }
        }

        stage('Build Maven') {
            steps {
                echo 'Building application with Maven...'
                script {
                    if (isUnix()) {
                        sh 'mvn clean package -DskipTests'
                    } else {
                        bat 'mvn clean package -DskipTests'
                    }
                }
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running unit tests...'
                script {
                    if (isUnix()) {
                        sh 'mvn test'
                    } else {
                        bat 'mvn test'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                script {
                    if (isUnix()) {
                        sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                        sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                    } else {
                        bat "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                        bat "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                    }
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo 'Pushing Docker image to DockerHub...'
                script {
                    if (isUnix()) {
                        sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
                        sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                        sh "docker push ${DOCKER_IMAGE}:latest"
                    } else {
                        bat "echo %DOCKERHUB_CREDENTIALS_PSW% | docker login -u %DOCKERHUB_CREDENTIALS_USR% --password-stdin"
                        bat "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                        bat "docker push ${DOCKER_IMAGE}:latest"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes...'
                script {
                    withCredentials([file(credentialsId: "${KUBECONFIG_CREDENTIAL}", variable: 'KUBECONFIG')]) {
                        if (isUnix()) {
                            sh """
                                export KUBECONFIG=\${KUBECONFIG}
                                kubectl set image deployment/crud-app-deployment \
                                crud-app=${DOCKER_IMAGE}:${DOCKER_TAG}
                                kubectl rollout status deployment/crud-app-deployment
                            """
                        } else {
                            bat """
                                set KUBECONFIG=%KUBECONFIG%
                                kubectl set image deployment/crud-app-deployment crud-app=${DOCKER_IMAGE}:${DOCKER_TAG}
                                kubectl rollout status deployment/crud-app-deployment
                            """
                        }
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Verifying deployment...'
                script {
                    withCredentials([file(credentialsId: "${KUBECONFIG_CREDENTIAL}", variable: 'KUBECONFIG')]) {
                        if (isUnix()) {
                            sh """
                                export KUBECONFIG=\${KUBECONFIG}
                                kubectl get pods -l app=crud-app
                                kubectl get services crud-app-service
                            """
                        } else {
                            bat """
                                set KUBECONFIG=%KUBECONFIG%
                                kubectl get pods -l app=crud-app
                                kubectl get services crud-app-service
                            """
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up...'
            script {
                if (isUnix()) {
                    sh 'docker logout || true'
                } else {
                    bat 'docker logout || exit 0'
                }
            }
        }
        success {
            echo '✓ Pipeline executed successfully!'
            echo "Application deployed with image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
        }
        failure {
            echo '✗ Pipeline failed! Check the logs for details.'
        }
    }
}
