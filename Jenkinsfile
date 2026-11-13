
pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                echo 'Pulling source code from GitHub'
                checkout scm
            }
        }

        stage('Code Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'

                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=jenkins-docker-poc
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Quality Gate failed: ${qg.status}"
                        }
                    }
                }
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image'

                sh """
                    docker build \
                    -t my-devops-app:${BUILD_NUMBER} .
                """
            }
        }

        stage('Docker Test') {
            steps {
                echo 'Testing Docker image'

                sh """
                    docker run --rm \
                    my-devops-app:${BUILD_NUMBER} \
                    nginx -t
                """
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application'

                sh """
                    docker stop my-devops-container || true
                    docker rm my-devops-container || true

                    docker run -d \
                    --name my-devops-container \
                    --restart unless-stopped \
                    -p 80:80 \
                    my-devops-app:${BUILD_NUMBER}
                """
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD Pipeline failed!'
        }
    }
}
