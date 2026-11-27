pipeline {
    agent any
    triggers {
        pollSCM('* * * * *')
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build and Test') {
            steps {
                sh 'mvn -B clean verify'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t leave-service:${BUILD_NUMBER} .'
            }
        }
        stage('Deploy to Staging') {
            steps {
                sh '''
                    docker rm -f leave-staging || true
                    docker run -d --name leave-staging -e APP_ENV=staging -p 8085:8080 leave-service:${BUILD_NUMBER}
                    sleep 5
                    curl -s http://localhost:8085
                '''
            }
        }
        stage('Approval') {
            steps {
                input message: 'Staging verified. Deploy to Production?', ok: 'Deploy'
            }
        }
        stage('Deploy to Production') {
            steps {
                sh '''
                    docker rm -f leave-prod || true
                    docker run -d --name leave-prod -e APP_ENV=production -p 8086:8080 leave-service:${BUILD_NUMBER}
                    sleep 5
                    curl -s http://localhost:8086
                    echo "--- Proving staging and production use the same image tag ---"
                    docker ps --filter name=leave-staging --filter name=leave-prod --format "table {{.Names}}\t{{.Image}}\t{{.Status}}"
                '''
            }
        }
    }
    post {
        success {
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }
    }
}
