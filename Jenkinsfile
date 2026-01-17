pipeline {
    agent any
    
    environment {
        SONAR_TOKEN = credentials('sonarqube-token') 
    }
    
    stages {
        stage('Preparation') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'docker compose build'
            }
        }
        
        stage('Trivy FS Scan') {
            steps {
                sh 'docker run --rm -v $WORKSPACE:/root/.cache/ aquasec/trivy fs . > trivy_report.txt'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'
                    // Thêm bước chờ để đảm bảo SonarQube API đã sẵn sàng
                    echo "Waiting for SonarQube to be fully ready..."
                    sleep 20 

                    withSonarQubeEnv('SonarQube') {
                        // Sử dụng IP 172.17.0.1 và tham số -Dsonar.token mới
                        sh "${scannerHome}/bin/sonar-scanner " +
                           "-Dsonar.projectKey=microservice-app " +
                           "-Dsonar.sources=. " +
                           "-Dsonar.exclusions=**/users-api/** " +
                           "-Dsonar.host.url=http://172.17.0.1:9000 " +
                           "-Dsonar.token=${SONAR_TOKEN}"
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                sh 'docker compose down'
                sh 'docker compose up -d'
            }
        }
    }
}