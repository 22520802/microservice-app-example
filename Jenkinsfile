pipeline {
    agent any
    
    environment {
        // ID của credential đã tạo trong Jenkins
        SONAR_TOKEN = credentials('sonarqube-token') 
    }
    
    stages {
        stage('Preparation') {
            steps {
                echo 'Pulling code...'
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                script {
                   echo 'Building Docker Images...'
                   sh 'docker compose build'
                }
            }
        }
        
        stage('Trivy FS Scan') {
            steps {
                echo 'Scanning File System...'
                sh 'docker run --rm -v $WORKSPACE:/root/.cache/ aquasec/trivy fs . > trivy_report.txt'
                sh 'cat trivy_report.txt' 
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('SonarQube') {
                        // SỬA TẠI ĐÂY: Dùng dấu nháy đơn bao ngoài và nháy kép cho biến để đảm bảo token dính liền vào dấu bằng
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=microservice-app -Dsonar.sources=. -Dsonar.exclusions=**/users-api/** -Dsonar.host.url=http://sonarqube:9000 -Dsonar.login=${SONAR_TOKEN}"
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    echo 'Deploying Microservices...'
                    sh 'docker compose down'
                    sh 'docker compose up -d'
                }
            }
        }
    }

    post {
        success {
            echo 'Chúc mừng! Pipeline đã hoàn thành.'
        }
        failure {
            echo 'Pipeline thất bại. Kiểm tra lại tham số SonarQube.'
        }
    }
}