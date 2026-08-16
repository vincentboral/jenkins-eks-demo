pipeline {

    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '907480720480'
        ECR_REPOSITORY = 'jenkins-eks-demo'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_NAME = "${ECR_REGISTRY}/${ECR_REPOSITORY}"
        EKS_CLUSTER = 'jenkins-eks-cluster'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out application source code from GitHub...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t ${IMAGE_NAME}:latest .'
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                echo 'Logging in to Amazon ECR...'
                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} |
                    docker login --username AWS --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                echo 'Pushing Docker image to Amazon ECR...'
                sh 'docker push ${IMAGE_NAME}:latest'
            }
        }

        stage('Deploy to EKS') {
            steps {
                echo 'Configuring kubectl for EKS...'
                sh '''
                    aws eks update-kubeconfig \
                    --region ${AWS_REGION} \
                    --name ${EKS_CLUSTER} \
                    --kubeconfig /var/lib/jenkins/.kube/config
                '''

                echo 'Deploying application to EKS...'
                sh '''
                    KUBECONFIG=/var/lib/jenkins/.kube/config \
                    kubectl apply -f k8s-deployment.yaml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Checking Kubernetes deployment...'
                sh '''
                    KUBECONFIG=/var/lib/jenkins/.kube/config \
                    kubectl get deployment jenkins-eks-demo
                '''

                echo 'Checking application pods...'
                sh '''
                    KUBECONFIG=/var/lib/jenkins/.kube/config \
                    kubectl get pods -l app=jenkins-eks-demo
                '''

                echo 'Checking application service...'
                sh '''
                    KUBECONFIG=/var/lib/jenkins/.kube/config \
                    kubectl get service jenkins-eks-demo-service
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD pipeline failed. Check the Jenkins console output.'
        }
    }
}
