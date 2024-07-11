pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-hub-credentials'
        REPO_URL = 'https://github.com/Jagannathan88/capstone-project.git'
        BRANCH = 'refs/heads/dev'
        DOCKER_IMAGE = 'jagannathan88/dev:latest'
        CONTAINER_NAME = 'my-app-container'
    }

    stages {
        stage('Checkout') {
            steps {
                cleanWs()
                checkout([$class: 'GitSCM', branches: [[name: "${env.BRANCH}"]],
                          doGenerateSubmoduleConfigurations: false, extensions: [], 
                          submoduleCfg: [], userRemoteConfigs: [[url: "${env.REPO_URL}"]]])
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${env.DOCKER_HUB_CREDENTIALS}") {
                        def dockerImage = docker.build("${env.DOCKER_IMAGE}")
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy Container') {
            steps {
                script {
                    // Stop and remove the container if it exists
                    sh "docker ps -aqf name=${env.CONTAINER_NAME} | xargs -r docker stop || true"
                    sh "docker ps -aqf name=${env.CONTAINER_NAME} | xargs -r docker rm || true"

                    // Run the new container
                    sh "docker run -d -p 80:80 --name ${env.CONTAINER_NAME} ${env.DOCKER_IMAGE}"
                }
            }
        }
    }

    post {
        always {
            script {
                // Clean up stopped containers
                sh 'docker container prune -f || true'
                // Clean up unused images
                sh 'docker image prune -f || true'
                // Clean up dangling volumes
                sh 'docker volume prune -f || true'
                // Clean up dangling networks
                sh 'docker network prune -f || true'
            }
        }
    }
}

