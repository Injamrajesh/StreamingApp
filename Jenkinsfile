pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '310297108115'
        IMAGE_TAG = "${env.GIT_COMMIT}"

        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"

        FRONTEND_REPO = "${ECR_REGISTRY}/streamingapp-frontend"
        AUTH_REPO = "${ECR_REGISTRY}/streamingapp-auth-service"
        STREAMING_REPO = "${ECR_REGISTRY}/streamingapp-streaming-service"
        ADMIN_REPO = "${ECR_REGISTRY}/streamingapp-admin-service"
        CHAT_REPO = "${ECR_REGISTRY}/streamingapp-chat-service"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Images') {
            steps {
                bat 'docker build -t streamingapp-frontend:%IMAGE_TAG% ./frontend'
                bat 'docker build -t streamingapp-auth:%IMAGE_TAG% ./backend/authService'
                bat 'docker build -t streamingapp-streaming:%IMAGE_TAG% -f ./backend/streamingService/Dockerfile ./backend'
                bat 'docker build -t streamingapp-admin:%IMAGE_TAG% -f ./backend/adminService/Dockerfile ./backend'
                bat 'docker build -t streamingapp-chat:%IMAGE_TAG% -f ./backend/chatService/Dockerfile ./backend'
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-credentials-rajesh']
                ]) {
                    bat '''
                    aws ecr get-login-password --region %AWS_REGION% | docker login --username AWS --password-stdin %ECR_REGISTRY%
                    '''
                }
            }
        }

        stage('Tag Docker Images') {
            steps {
                bat 'docker tag streamingapp-frontend:%IMAGE_TAG% %FRONTEND_REPO%:%IMAGE_TAG%'
                bat 'docker tag streamingapp-auth:%IMAGE_TAG% %AUTH_REPO%:%IMAGE_TAG%'
                bat 'docker tag streamingapp-streaming:%IMAGE_TAG% %STREAMING_REPO%:%IMAGE_TAG%'
                bat 'docker tag streamingapp-admin:%IMAGE_TAG% %ADMIN_REPO%:%IMAGE_TAG%'
                bat 'docker tag streamingapp-chat:%IMAGE_TAG% %CHAT_REPO%:%IMAGE_TAG%'
            }
        }

        stage('Push Images to ECR') {
            steps {
                bat 'docker push %FRONTEND_REPO%:%IMAGE_TAG%'
                bat 'docker push %AUTH_REPO%:%IMAGE_TAG%'
                bat 'docker push %STREAMING_REPO%:%IMAGE_TAG%'
                bat 'docker push %ADMIN_REPO%:%IMAGE_TAG%'
                bat 'docker push %CHAT_REPO%:%IMAGE_TAG%'
            }
        }
    }

    post {
        success {
            echo 'StreamingApp CI/CD pipeline completed successfully.'
        }

        failure {
            echo 'StreamingApp CI/CD pipeline failed.'
        }
    }
}