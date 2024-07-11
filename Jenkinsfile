pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-hub-credentials'
        REPO_URL = 'https://github.com/Jagannathan88/capstone-project.git'
        DEV_DOCKER_IMAGE = 'jagannathan88/dev:latest'
        PROD_DOCKER_IMAGE = 'jagannathan88/prod:latest'
        CONTAINER_NAME = 'my-app-container'
    }

    stages {
        stage('Checkout') {
            steps {
                cleanWs()
                script {
                    def branch = env.GIT_BRANCH ?: 'dev' // default to 'dev' if GIT_BRANCH is not set
                    echo "Branch: ${branch}"
                    git branch: "${branch}", url: "${env.REPO_URL}"
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    if (env.GIT_BRANCH == 'dev') {
                        // Build and push to dev repository
                        docker.withRegistry('https://index.docker.io/v1/', "${env.DOCKER_HUB_CREDENTIALS}") {
                            def image = docker.build(env.DEV_DOCKER_IMAGE)
                            image.push()
                        }
                    } else if (env.GIT_BRANCH == 'master') {
                        // Build and push to prod repository
                        docker.withRegistry('https://index.docker.io/v1/', "${env.DOCKER_HUB_CREDENTIALS}") {
                            def image = docker.build(env.PROD_DOCKER_IMAGE)
                            image.push()
                        }
                    } else {
                        error "Unknown branch: ${env.GIT_BRANCH}"
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
                    def dockerImage = ''
                    if (env.GIT_BRANCH == 'dev') {
                        dockerImage = env.DEV_DOCKER_IMAGE
                    } else if (env.GIT_BRANCH == 'master') {
                        dockerImage = env.PROD_DOCKER_IMAGE
                    }
                    sh "docker run -d -p 80:80 --name ${env.CONTAINER_NAME} ${dockerImage}"
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

