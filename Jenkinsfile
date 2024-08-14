pipeline {
    agent any 
    environment {
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        ECR_REPO = '266852548854.dkr.ecr.ap-northeast-2.amazonaws.com/test_jenkins'
        REGION = 'ap-northeast-2'
        CLUSTER_NAME = 'test-eks-cluster'
        DEPLOYMENT_NAME = 'deploybyjenkins'
        CONTAINER_NAME = 'flaskapp'
        KUBECONFIG = '/home/ubuntu/config'  // 현재 config 파일 경로로 수정
    }
    
    stages { 
        stage('Build docker image') {
            steps {  
                sh 'docker build -t $ECR_REPO:$BUILD_NUMBER .'
            }
        }
        stage('login to ECR') {
            steps {
                sh 'aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $ECR_REPO'
            }
        }
        stage('Push image to ECR') {
            steps {
                sh 'docker push $ECR_REPO:$BUILD_NUMBER'
            }
        }
        stage('Deploy to EKS') {
            steps {
                sh '''
                    whoami
                    aws eks update-kubeconfig --name test-eks-cluster
                    kubectl apply -f deployment.yaml
                '''
            }
        }
    }
    post {
        always {
            sh 'docker logout'
        }
    }
}
