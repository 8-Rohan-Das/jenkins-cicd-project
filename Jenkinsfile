pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-2'
        AWS_ACCOUNT_ID = '510821107413'
        ECR_REPO = 'jenkins-cicd-project'
        IMAGE_TAG = "v${BUILD_NUMBER}"
        IMAGE_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}"
    }

    stages {

        stage('Clone Repository') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${ECR_REPO}:${IMAGE_TAG} .'
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region ${AWS_REGION} | \
                docker login --username AWS --password-stdin \
                ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                '''
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh '''
                docker tag ${ECR_REPO}:${IMAGE_TAG} ${IMAGE_URI}
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                docker push ${IMAGE_URI}
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl set image deployment/react-app react-app=${IMAGE_URI}
                kubectl rollout status deployment/react-app
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline executed successfully!'
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}