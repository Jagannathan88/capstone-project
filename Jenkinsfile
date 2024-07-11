pipeline {
    agent any

    environment {
        // Define Docker Hub repositories
        DEV_REPO = 'jagannathan88/dev:latest'
        PROD_REPO = 'jagannathan88/production:latest'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the code from GitHub repository
                checkout([$class: 'GitSCM',
                          branches: [[name: "refs/heads/${BRANCH}"]],
                          doGenerateSubmoduleConfigurations: false,
                          extensions: [[$class: 'CleanBeforeCheckout']],
                          submoduleCfg: [],
                          userRemoteConfigs: [[credentialsId: 'github-credentials',
                                               url: 'https://github.com/Jagannathan88/capstone-project.git']]])
            }
        }

        stage('Build and Push Docker Image') {
            when {
                branch 'dev'
            }
            steps {
                script {
                    def dockerImage = docker.build(DEV_REPO)
                    dockerImage.push()
                }
            }
        }

        stage('Deploy to Production') {
            when {
                branch 'master'
            }
            steps {
                script {
                    def dockerImage = docker.build(PROD_REPO)
                    dockerImage.push()
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}

