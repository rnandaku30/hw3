pipeline {
    agent any
    
    tools {
        maven 'Maven 3.8.1'
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('rams-docker-pass') // Docker credentials added to Jenkins
    }
    stages {
        stage('Initialize') {
            steps {
                script {
                    // Defining a build timestamp variable
                    env.BUILD_TIMESTAMP = new Date().format("yyyyMMddHHmmss", TimeZone.getTimeZone('UTC'))
                    echo "Build timestamp: ${env.BUILD_TIMESTAMP}"
                }
            }
        }

        stage('Prepare Buildx') {
            steps {
                script {
                    // Install and enable Docker Buildx
                    sh """
                        mkdir -p ~/.docker/cli-plugins
                        curl -SL https://github.com/docker/buildx/releases/latest/download/buildx-linux-amd64 -o ~/.docker/cli-plugins/docker-buildx
                        chmod +x ~/.docker/cli-plugins/docker-buildx
                        docker buildx version
                        docker buildx create --use
                    """
                }
            }
        }

        stage('Building the Application Image') {
            steps {
                script {
                    // Checkout SCM
                    checkout scm

                    // Change directory to 'demo' for Maven build
                    sh 'mvn clean package'

                    // Securely handling Docker login
                    withCredentials([usernamePassword(credentialsId: 'rams-docker-pass',
                                                      usernameVariable: 'DOCKER_USER',
                                                      passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USER" --password-stdin
                        """
                    }

                    // Building Docker image for multiple platforms using Buildx
                    def imageName = "rnandaku30/studentsapp:${env.BUILD_TIMESTAMP}"
                    sh "docker buildx build --platform linux/amd64,linux/arm64 -t ${imageName} --push ."

                    // Save image name for later stages
                    env.IMAGE_NAME = imageName
                }
            }
        }

        stage('Deploying to Rancher') {
            steps {
                script {
                    // Deploying the new image to Rancher
                    sh "kubectl set image deployment/hw3-deployment container-0=${env.IMAGE_NAME}"
                }
            }
        }
    }
}