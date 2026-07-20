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

        stage('SonarQube Code Analysis') {
            steps {
                echo 'Running SonarQube Scanner for PERN Store...'
                script {
                    def scannerHome = tool 'sonar-scanner'
                    withSonarQubeEnv('SonarQube-Server') {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }

        stage('SonarQube Quality Gate') {
            steps {
                echo 'Checking SonarQube Quality Gate...'
                script {
                    timeout(time: 5, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to Quality Gate failure: ${qg.status}"
                        }
                    }
                }
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

        stage('Trivy Security Scan') {
            steps {
                echo 'Scanning Frontend & Backend Docker Images with Trivy...'
                sh "trivy image --severity HIGH,CRITICAL ${env.DOCKER_HUB_USER}/pern-frontend:latest"
                sh "trivy image --severity HIGH,CRITICAL ${env.DOCKER_HUB_USER}/pern-backend:latest"
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


        stage('Setup Environment Files') {
            steps {
                withCredentials([
                    file(credentialsId: 'server-env-file', variable: 'SERVER_ENV'),
                    file(credentialsId: 'client-env-file', variable: 'CLIENT_ENV')
                ]) {

                    sh 'rm -rf server/.env client/.env'
                    sh 'cp $SERVER_ENV server/.env'
                    sh 'cp $CLIENT_ENV client/.env'
                 }
             }
        }

        stage('5. Deploy / Run Containers') {
            steps {
                echo 'Running containers using Docker Compose...'

                sh 'docker compose down -v'
                sh 'docker compose up --build -d'
                
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
            echo 'FAILURE: Sending alert email to developer...'
            mail to: 'ektagupta2004v@gmail.com',
                subject: "ALERT: Jenkins Pipeline Failed! - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """ALERT: Jenkins Pipeline Failed!

                    Your pipeline has crashed. The biggest reason for this is that your code quality is not up to the mark. Please check the SonarQube dashboard for more details.

                    --------------------------------------------------
                    Build Details:
                    - Pipeline Name : ${env.JOB_NAME}
                    - Build Number  : #${env.BUILD_NUMBER}
                    - Build URL     : ${env.BUILD_URL}
                    - SonarQube     : http://localhost:9000
                    --------------------------------------------------

                    Please check the logs and fix the issues."""
        }
    }
}