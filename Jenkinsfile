pipeline {

    agent any

    environment {

        AWS_REGION = "us-east-1"

        ACCOUNT_ID = sh(
            script: "aws sts get-caller-identity --query Account --output text",
            returnStdout: true
        ).trim()

        ECR_REPO = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/nginx-app"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build \
                    -t nginx-app:${BUILD_NUMBER} \
                    -t nginx-app:latest .
                '''
            }
        }

        stage('Login ECR') {
            steps {
                sh '''
                aws ecr get-login-password \
                    --region ${AWS_REGION} \
                | docker login \
                    --username AWS \
                    --password-stdin ${ECR_REPO}
                '''
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                docker tag nginx-app:${BUILD_NUMBER} \
                    ${ECR_REPO}:${BUILD_NUMBER}

                docker tag nginx-app:latest \
                    ${ECR_REPO}:latest

                docker push ${ECR_REPO}:${BUILD_NUMBER}
                docker push ${ECR_REPO}:latest
                '''
            }
        }

        stage('Start Instance Refresh') {
            steps {
                sh '''
                aws autoscaling start-instance-refresh \
                    --auto-scaling-group-name nginx-asg
                '''
            }
        }

    }
}