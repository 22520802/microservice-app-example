pipeline {
    agent any
    
    environment {
        // ID của credential đã tạo trong Jenkins
        SONAR_TOKEN = credentials('sonarqube-token') 
    }
    
    stages {
        stage('Preparation') {
            steps {
                // Clone code 
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                script {
                   echo 'Building Docker Images...'
                   // Build app [cite: 608]
                   sh 'docker compose build'
                }
            }
        }
        
        stage('Trivy FS Scan') {
            steps {
                // Quét bảo mật filesystem [cite: 610]
                // Dùng docker chạy trivy để không phải cài đặt trivy lên máy host
                sh 'docker run --rm -v $WORKSPACE:/root/.cache/ aquasec/trivy fs . > trivy_report.txt'
                sh 'cat trivy_report.txt' // In kết quả ra log
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                script {
                    // Phân tích code [cite: 611]
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('SonarQube') {
                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=microservice-app \
                        -Dsonar.sources=. \
                        -Dsonar.exclusions=**/users-api/** \
                        -Dsonar.host.url=http://sonarqube:9000 \
                        -Dsonar.login=$SONAR_TOKEN
                        """
                        // Lưu ý: users-api bị exclude theo báo cáo [cite: 612]
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    echo 'Deploying...'
                    // Deploy app 
                    sh 'docker compose down'
                    sh 'docker compose up -d'
                }
            }
        }
    }
}