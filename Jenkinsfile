pipeline {
    agent any
    
    environment {
        // ID của credential đã tạo trong Jenkins
        SONAR_TOKEN = credentials('sonarqube-token') 
    }
    
    stages {
        stage('Preparation') {
            steps {
                echo 'Pulling code from Repository...'
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                script {
                   echo 'Building Docker Images...'
                   // Sử dụng docker compose build để tạo các images từ Dockerfile đã sửa
                   sh 'docker compose build'
                }
            }
        }
        
        stage('Trivy FS Scan') {
            steps {
                echo 'Scanning File System for security vulnerabilities...'
                // Quét bảo mật filesystem và lưu báo cáo
                sh 'docker run --rm -v $WORKSPACE:/root/.cache/ aquasec/trivy fs . > trivy_report.txt'
                sh 'cat trivy_report.txt' 
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('SonarQube') {
                        // VIẾT LIỀN TRÊN 1 DÒNG để tránh lỗi "Unrecognized option" do khoảng trắng/nối chuỗi
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=microservice-app -Dsonar.sources=. -Dsonar.exclusions=**/users-api/** -Dsonar.host.url=http://sonarqube:9000 -Dsonar.login=${SONAR_TOKEN}"
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    echo 'Deploying Microservices...'
                    // Dừng các container cũ và khởi chạy bản mới nhất
                    sh 'docker compose down'
                    sh 'docker compose up -d'
                    echo 'Application is running at http://<Your-EC2-IP>:8081'
                }
            }
        }
    }

    post {
        success {
            echo 'Chúc mừng! Toàn bộ Pipeline đã hoàn thành thành công.'
        }
        failure {
            echo 'Pipeline thất bại. Vui lòng kiểm tra Console Output để xem chi tiết lỗi.'
        }
    }
}