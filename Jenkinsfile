pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-credentials')
        REPO_URL = 'https://github.com/Jagannathan88/capstone-project.git'
        DEV_BRANCH = 'dev'
        MASTER_BRANCH = 'master'
        DEV_DOCKER_IMAGE = 'jagannathan88/dev:latest'
        PROD_DOCKER_IMAGE = 'jagannathan88/prod:latest'
        CONTAINER_NAME = 'my-app-container'
    }

    stages {
        stage('Checkout') {
            steps {
                cleanWs()
                script {
                    sh "git clone ${REPO_URL}"
                    sh "cd capstone-project && git checkout ${DEV_BRANCH}"
                }
            }
        }

        stage('Build and Push Docker Image') {
            when {
                branch "${DEV_BRANCH}"
            }
            steps {
                echo "Building and pushing Docker image to development repository"
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_HUB_CREDENTIALS}") {
                        def dockerImage = docker.build("${DEV_DOCKER_IMAGE}")
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy Container') {
            when {
                branch "${DEV_BRANCH}"
            }
            steps {
                echo "Deploying container from development image"
                script {
                    // Stop and remove the container if it exists
                    sh "docker ps -aqf name=${CONTAINER_NAME} | xargs -r docker stop || true"
                    sh "docker ps -aqf name=${CONTAINER_NAME} | xargs -r docker rm || true"

                    // Run the new container
                    sh "docker run -d -p 80:80 --name ${CONTAINER_NAME} ${DEV_DOCKER_IMAGE}"
                }
            }
        }

        stage('Push to Prod Repository') {
            when {
                branch "${MASTER_BRANCH}"
            }
            steps {
                echo "Pushing Docker image to production repository"
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_HUB_CREDENTIALS}") {
                        def dockerImage = docker.build("${PROD_DOCKER_IMAGE}")
                        dockerImage.push()
                    }
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

