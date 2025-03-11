pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = "<AWS_ACCOUNT_ID>"
        AWS_REGION = "us-east-1"
        ECR_REPO = "three-tier-microservices"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        KUBE_CONFIG_PATH = "/var/lib/jenkins/.kube/config"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github-ssh-key', url: 'git@github.com:lokesh-mateti/three-tier-architecture-demo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                    docker build -t $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG -f cart/Dockerfile .
                    '''
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    sh '''
                    aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                    docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    sh '''
                    kubectl set image deployment/cart-deployment cart=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG --kubeconfig $KUBE_CONFIG_PATH
                    kubectl rollout status deployment/cart-deployment --kubeconfig $KUBE_CONFIG_PATH
                    '''
                }
            }
        }
    }
}
