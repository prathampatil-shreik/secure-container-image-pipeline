pipeline {
    agent any

    environment {
        IMAGE_NAME = "loan-calculator"
        IMAGE_TAG  = "${BUILD_NUMBER}"
        DOCKER_HUB_REPO = "your-dockerhub-username/${IMAGE_NAME}"
    }

    tools {
        maven 'Maven-3.9'
        jdk   'JDK-17'
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
                dir('loan-calculator') {
                    sh 'mvn clean package -DskipTests -q'
                }
            }
        }

        stage('Test') {
            steps {
                dir('loan-calculator') {
                    sh 'mvn test'
                }
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'loan-calculator/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Docker Build') {
            steps {
                dir('loan-calculator') {
                    sh "docker build -t ${DOCKER_HUB_REPO}:${IMAGE_TAG} ."
                    sh "docker tag ${DOCKER_HUB_REPO}:${IMAGE_TAG} ${DOCKER_HUB_REPO}:latest"
                }
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                    sh "docker push ${DOCKER_HUB_REPO}:${IMAGE_TAG}"
                    sh "docker push ${DOCKER_HUB_REPO}:latest"
                }
            }
        }

        stage('Deploy') {
            steps {
                sh """
                    docker stop ${IMAGE_NAME} || true
                    docker rm   ${IMAGE_NAME} || true
                    docker run -d --name ${IMAGE_NAME} -p 8080:8080 ${DOCKER_HUB_REPO}:latest
                """
            }
        }
    }

    post {
        success { echo "Pipeline completed successfully. App running on port 8080." }
        failure { echo "Pipeline failed. Check the logs above." }
        always  { sh "docker logout || true" }
    }
}
