pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'ekta151'
        DOCKER_CRED_ID  = 'docker-hub-credentials'
        
        PATH            = "/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/opt/homebrew/bin:${env.PATH}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo ' Step 1: Fetching code using Jenkins UI SCM configuration...'
                checkout scm
            }
        }

        stage('Build Frontend Image') {
            steps {
                echo 'Step 2: Building PERN Store Client (Frontend) Docker Image...'
                dir('client') {
                    sh "docker build -t ${env.DOCKER_HUB_USER}/pern-frontend:latest ."
                }
            }
        }

        stage('Build Backend Image') {
            steps {
                echo 'Step 3: Building PERN Store Server (Backend) Docker Image...'
                dir('server') {
                    sh "docker build -t ${env.DOCKER_HUB_USER}/pern-backend:latest ."
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo 'Step 4: Logging into Docker Hub and pushing fresh images...'
                withCredentials([usernamePassword(
                    credentialsId: "${env.DOCKER_CRED_ID}", 
                    usernameVariable: 'DOCKER_USER', 
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                    
                    echo 'Pushing Frontend Image...'
                    sh "docker push ${env.DOCKER_HUB_USER}/pern-frontend:latest"
                    
                    echo 'Pushing Backend Image...'
                    sh "docker push ${env.DOCKER_HUB_USER}/pern-backend:latest"
                }
            }
        }

        stage('5. Deploy / Run Containers') {
            steps {
                echo 'Running containers using Docker Compose...'
                
                sh "docker compose down"
                sh "docker compose up -d"
                
                echo ' Application is now live and running!'
            }
        }
    }

    post {
        always {
            echo 'Cleaning up local session tokens...'
            sh "docker logout"
        }
        success {
            echo 'SUCCESS: Pipeline built, pushed, and deployed perfectly!'
        }
        failure {
            echo 'FAILURE: Check Jenkins Console Output.'
        }
    }
}