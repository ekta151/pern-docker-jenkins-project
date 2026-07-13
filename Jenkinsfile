pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'ekta151'
        DOCKER_CRED_ID  = 'docker-hub-credentials'
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo '🚚 Step 1: Fetching code using Jenkins UI SCM configuration...'
                checkout scm
            }
        }

        stage('Build Frontend Image') {
            steps {
                echo '🏗️ Step 2: Building PERN Store Client (Frontend) Docker Image...'
                dir('client') {
                    sh "/usr/local/bin/docker build -t ${env.DOCKER_HUB_USER}/pern-frontend:latest ."
                }
            }
        }

        stage('Build Backend Image') {
            steps {
                echo '🏗️ Step 3: Building PERN Store Server (Backend) Docker Image...'
                dir('server') {
                    sh "/usr/local/bin/docker build -t ${env.DOCKER_HUB_USER}/pern-backend:latest ."
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo '🚀 Step 4: Logging into Docker Hub and pushing fresh images...'
                withCredentials([usernamePassword(
                    credentialsId: "${env.DOCKER_CRED_ID}", 
                    usernameVariable: 'DOCKER_USER', 
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh "echo \$DOCKER_PASS | /usr/local/bin/docker login -u \$DOCKER_USER --password-stdin"
                    
                    echo 'Pushing Frontend Image...'
                    sh "/usr/local/bin/docker push ${env.DOCKER_HUB_USER}/pern-frontend:latest"
                    
                    echo 'Pushing Backend Image...'
                    sh "/usr/local/bin/docker push ${env.DOCKER_HUB_USER}/pern-backend:latest"
                }
            }
        }
    }

    post {
        always {
            echo '🧹 Cleaning up local session tokens...'
            // Full path for docker logout
            sh "/usr/local/bin/docker logout"
        }
        success {
            echo '🎉 SUCCESS: Pipeline finished perfectly! Check your Docker Hub account.'
        }
        failure {
            echo '❌ FAILURE: Oops, koi step toot gaya. Check the Jenkins Console Output.'
        }
    }
}