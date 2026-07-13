pipeline {
    agent any

    environment {
        // Apne Docker Hub ka username yahan set karo
        DOCKER_HUB_USER = 'ekta151'
        
        // Jenkins Credentials ID jo humne Docker Hub ke liye banaya hai
        DOCKER_CRED_ID  = 'docker-hub-credentials'
    }

    stages {
        // 1. SCM SE AUTOMATIC CODE FETCH KARNA
        stage('Checkout Code') {
            steps {
                echo '🚚 Step 1: Fetching code using Jenkins UI SCM configuration...'
                // Yeh single-line tumhaare Jenkins dashboard ki settings ko use karegi
                checkout scm
            }
        }

        // 2. FRONTEND DOCKER IMAGE BUILD KARNA
        stage('Build Frontend Image') {
            steps {
                echo '🏗️ Step 2: Building PERN Store Client (Frontend) Docker Image...'
                dir('client') {
                    sh "docker build -t ${env.DOCKER_HUB_USER}/pern-frontend:latest ."
                }
            }
        }

        // 3. BACKEND DOCKER IMAGE BUILD KARNA
        stage('Build Backend Image') {
            steps {
                echo '🏗️ Step 3: Building PERN Store Server (Backend) Docker Image...'
                dir('server') {
                    sh "docker build -t ${env.DOCKER_HUB_USER}/pern-backend:latest ."
                }
            }
        }

        // 4. DOCKER HUB PAR PUSH KARNA
        stage('Push to Docker Hub') {
            steps {
                echo '🚀 Step 4: Logging into Docker Hub and pushing fresh images...'
                
                // Securely username and password ko env variables mein inject karna
                withCredentials([usernamePassword(
                    credentialsId: "${env.DOCKER_CRED_ID}", 
                    usernameVariable: 'DOCKER_USER', 
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    // Password secure tarike se stdin ke raste login mein pass hoga
                    sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                    
                    echo 'Pushing Frontend Image...'
                    sh "docker push ${env.DOCKER_HUB_USER}/pern-frontend:latest"
                    
                    echo 'Pushing Backend Image...'
                    sh "docker push ${env.DOCKER_HUB_USER}/pern-backend:latest"
                }
            }
        }
    }

    // PIPELINE KE BAAD KA STATUS ORE CLEANUP
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