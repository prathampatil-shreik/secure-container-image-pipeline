pipeline {
    agent any

    environment {
        IMAGE_NAME      = "loan-calculator"
        IMAGE_TAG       = "${BUILD_NUMBER}"
        DOCKER_HUB_REPO = "pratham20021/loan-calculator"
    }

    tools {
        maven 'Default Maven'
        jdk   'JDK17'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/pratham20021/loan-calculator.git']]
                ])
            }
        }

        stage('Build') {
            steps {
                bat 'mvn clean package -DskipTests -q'
            }
        }

        stage('Test') {
            steps {
                bat 'mvn test'
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Docker Build') {
            steps {
                bat "docker build -t %DOCKER_HUB_REPO%:%IMAGE_TAG% ."
                bat "docker tag %DOCKER_HUB_REPO%:%IMAGE_TAG% %DOCKER_HUB_REPO%:latest"
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat "docker login -u %DOCKER_USER% -p %DOCKER_PASS%"
                    bat "docker push %DOCKER_HUB_REPO%:%IMAGE_TAG%"
                    bat "docker push %DOCKER_HUB_REPO%:latest"
                }
            }
        }

        stage('Deploy') {
            steps {
                bat """
                    docker stop %IMAGE_NAME% || exit 0
                    docker rm   %IMAGE_NAME% || exit 0
                    docker run -d --name %IMAGE_NAME% -p 8080:8080 %DOCKER_HUB_REPO%:latest
                """
            }
        }
    }

    post {
        success { echo "Pipeline completed successfully. App running on port 8080." }
        failure { echo "Pipeline failed. Check the logs above." }
        always  { bat "docker logout || exit 0" }
    }
}
