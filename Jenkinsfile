pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = "ap-south-1"
        REPO_NAME = "pipeline"
        IMAGE_TAG = "latest"
        AWS_ACCOUNT_ID = "388762879277"
    }

    stages {
        stage('Clone') {
            steps {
                git 'https://github.com/SRIXENO/NEW-TRY.git'
            }
        }

        stage('Test') {
            steps {
                sh 'echo "No tests for static HTML, skipping..."'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $REPO_NAME:$IMAGE_TAG .
                docker tag $REPO_NAME:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$REPO_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Login to ECR & Push') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_DEFAULT_REGION \
                | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
                docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$REPO_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy to ECS') {
            steps {
                sh '''
                aws ecs update-service \
                  --cluster pipeline \
                  --service pipeline \
                  --force-new-deployment \
                  --region $AWS_DEFAULT_REGION
                '''
            }
        }
    }
}
