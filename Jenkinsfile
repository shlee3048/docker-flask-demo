pipeline {
    agent any 
    environment {
        AWS_CREDENTIALS = credentials('aws-credentials') // AWS 자격 증명
        ECR_REPO = '266852548854.dkr.ecr.ap-northeast-2.amazonaws.com/test_jenkins' // ECR 레포지토리 URI
        REGION = 'ap-northeast-2' // ECR과 EKS의 리전
        CLUSTER_NAME = 'test-eks-cluster' // EKS 클러스터 이름
        DEPLOYMENT_NAME = 'deploybyjenkins' // Kubernetes Deployment 이름
        CONTAINER_NAME = 'flaskapp' // Kubernetes Deployment에서의 컨테이너 이름
        KUBECONFIG = '/home/ssm-user/.kube/config' // SSM User의 Kubeconfig 경로
    }
    
    stages { 

        stage('Build docker image') {
            steps {  
                sh 'docker build -t $ECR_REPO:$BUILD_NUMBER .'
            }
        }
        stage('login to ECR') {
            steps{
                // 올바른 ECR 레지스트리 URL 사용
                sh 'aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $ECR_REPO'
            }
        }
        stage('Push image to ECR') {
            steps{
                sh 'docker push $ECR_REPO:$BUILD_NUMBER'
            }
        }
        stage('Deploy to EKS') {
            steps {
                // Kubernetes Deployment 업데이트
                sh '''
                    /usr/local/bin/kubectl set image deployment/$DEPLOYMENT_NAME $CONTAINER_NAME=$ECR_REPO:$BUILD_NUMBER --kubeconfig=/home/ssm-user/.kube/config
                    /usr/local/bin/kubectl rollout status deployment/$DEPLOYMENT_NAME --kubeconfig=/home/ssm-user/.kube/config
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
