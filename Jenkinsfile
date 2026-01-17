pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sonarqube-token')
    }

    stages {
        stage('Preparation') {
            steps {
                git url: "https://github.com/22520802/microservice-app-example.git", branch: 'master'
            }
        }

        stage('Build') {
            steps {
                echo 'Building...'
                // Simulate build process
                sh 'docker-compose build'
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs --exit-code 0 --severity MEDIUM,HIGH,CRITICAL . > trivy_report.txt 2>&1'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    echo "Starting SonarQube analysis..."
                    def scannerHome = tool 'SonarScanner' 
                    withSonarQubeEnv("SonarQube") {
                    sh "echo Using scanner at: ${scannerHome}"
                    sh "${scannerHome}/bin/sonar-scanner -Dsonar.login=$SONAR_TOKEN \
                    -Dsonar.exclusions=**/users-api/**"
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying...'
                sh 'docker-compose down'
                sh 'docker-compose up -d'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
