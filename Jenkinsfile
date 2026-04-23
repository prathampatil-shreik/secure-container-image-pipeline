pipeline {
    agent any

    tools {
        maven 'Default Maven'
        jdk   'JDK17'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                bat 'mvn clean package -DskipTests -q'
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat "docker build -t %DOCKER_USER%/loan-calculator:latest ."
                    bat "docker login -u %DOCKER_USER% -p %DOCKER_PASS%"
                    bat "docker push %DOCKER_USER%/loan-calculator:latest"
                }
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat "docker stop loan-calculator || exit 0"
                    bat "docker rm loan-calculator || exit 0"
                    bat "docker run -d --name loan-calculator -p 8080:8080 %DOCKER_USER%/loan-calculator:latest"
                }
            }
        }
    }

    post {
        success { echo "Pipeline succeeded. App running on port 8080." }
        failure { echo "Pipeline failed." }
    }
}
