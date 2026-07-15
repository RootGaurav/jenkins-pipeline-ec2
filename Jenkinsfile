

pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ROLE_ARN = 'arn:aws:iam::487916111349:role/jenkins-build-role'
        ECR_REPO = '487916111349.dkr.ecr.us-east-1.amazonaws.com/nginx-app-ecr'
    }


    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Assume Build Role') {
            steps {
                script {
                    def creds = sh(
                        script: """
                        aws sts assume-role \
                          --role-arn ${ROLE_ARN} \
                          --role-session-name build-session \
                          --query 'Credentials.[AccessKeyId,SecretAccessKey,SessionToken]' \
                          --output text
                        """,
                        returnStdout: true
                    ).trim().split()

                    env.AWS_ACCESS_KEY_ID = creds[0]
                    env.AWS_SECRET_ACCESS_KEY = creds[1]
                    env.AWS_SESSION_TOKEN = creds[2]
                }
            }
        }

        stage('Build Image') {
            steps {
                sh '''
                docker build \
                  -t nginx-app:${BUILD_NUMBER} \
                  -t nginx-app:latest .
                '''
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                aws ecr get-login-password \
                  --region ${AWS_REGION} \
                | docker login \
                  --username AWS \
                  --password-stdin ${ECR_REPO}

                docker tag nginx-app:${BUILD_NUMBER} \
                  ${ECR_REPO}:${BUILD_NUMBER}

                docker tag nginx-app:latest \
                  ${ECR_REPO}:latest

                docker push ${ECR_REPO}:${BUILD_NUMBER}
                docker push ${ECR_REPO}:latest
                '''
            }
        }
    }
}