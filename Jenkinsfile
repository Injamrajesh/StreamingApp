pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '310297108115'

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

                script {
                    env.IMAGE_TAG = sh(
                        script: 'git rev-parse --short=7 HEAD',
                        returnStdout: true
                    ).trim()

                    echo "Building commit: ${env.IMAGE_TAG}"
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
    docker build \
      --build-arg REACT_APP_AUTH_API_URL=/api \
      --build-arg REACT_APP_STREAMING_API_URL=/api \
      --build-arg REACT_APP_STREAMING_PUBLIC_URL= \
      --build-arg REACT_APP_ADMIN_API_URL=/api/admin \
      --build-arg REACT_APP_CHAT_API_URL=/api/chat \
      --build-arg REACT_APP_CHAT_SOCKET_URL=/ \
      -t streamingapp-frontend:${IMAGE_TAG} ./frontend
'''
                sh 'docker build -t streamingapp-auth:${IMAGE_TAG} ./backend/authService'
                sh 'docker build -t streamingapp-streaming:${IMAGE_TAG} -f ./backend/streamingService/Dockerfile ./backend'
                sh 'docker build -t streamingapp-admin:${IMAGE_TAG} -f ./backend/adminService/Dockerfile ./backend'
                sh 'docker build -t streamingapp-chat:${IMAGE_TAG} -f ./backend/chatService/Dockerfile ./backend'
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-credentials-rajesh']
                ]) {
                    sh '''
                        aws ecr get-login-password --region ${AWS_REGION} |
                        docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    '''
                }
            }
        }

        stage('Tag Docker Images') {
            steps {
                sh 'docker tag streamingapp-frontend:${IMAGE_TAG} ${FRONTEND_REPO}:${IMAGE_TAG}'
                sh 'docker tag streamingapp-auth:${IMAGE_TAG} ${AUTH_REPO}:${IMAGE_TAG}'
                sh 'docker tag streamingapp-streaming:${IMAGE_TAG} ${STREAMING_REPO}:${IMAGE_TAG}'
                sh 'docker tag streamingapp-admin:${IMAGE_TAG} ${ADMIN_REPO}:${IMAGE_TAG}'
                sh 'docker tag streamingapp-chat:${IMAGE_TAG} ${CHAT_REPO}:${IMAGE_TAG}'
            }
        }

        stage('Push Images to ECR') {
            steps {
                sh 'docker push ${FRONTEND_REPO}:${IMAGE_TAG}'
                sh 'docker push ${AUTH_REPO}:${IMAGE_TAG}'
                sh 'docker push ${STREAMING_REPO}:${IMAGE_TAG}'
                sh 'docker push ${ADMIN_REPO}:${IMAGE_TAG}'
                sh 'docker push ${CHAT_REPO}:${IMAGE_TAG}'
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