pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'ekta151'
        DOCKER_CRED_ID  = 'docker-hub-credentials'
        
        // 🔥 YEH LINE SABSE IMPORTANT HAI: Isse Jenkins ko MacBook ke saare standard binary paths mil jayenge
        PATH            = "/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/opt/homebrew/bin:${env.PATH}"
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
                    // Ab direct 'docker' command chalegi bina full path ke
                    sh "docker build -t ${env.DOCKER_HUB_USER}/pern-frontend:latest ."
                }
            }
        }

        stage('Build Backend Image') {
            steps {
                echo '🏗️ Step 3: Building PERN Store Server (Backend) Docker Image...'
                dir('server') {
                    sh "docker build -t ${env.DOCKER_HUB_USER}/pern-backend:latest ."
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
                    sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                    
                    echo 'Pushing Frontend Image...'
                    sh "docker push ${env.DOCKER_HUB_USER}/pern-frontend:latest"
                    
                    echo 'Pushing Backend Image...'
                    sh "docker push ${env.DOCKER_HUB_USER}/pern-backend:latest"
                }
            }
        }
    }

    post {
        always {
            echo '🧹 Cleaning up local session tokens...'
            sh "docker logout"
        }
        success {
            echo '🎉 SUCCESS: Pipeline finished perfectly! Check your Docker Hub account.'
        }
        failure {
            echo '❌ FAILURE: Oops, koi step toot gaya. Check the Jenkins Console Output.'
        }
    }
}