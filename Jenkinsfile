pipeline {
    agent any 
    environment {
        AWS_CREDENTIALS = credentials('aws-credentials')
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
                    sudo /usr/local/bin/kubectl set image deployment/$DEPLOYMENT_NAME $CONTAINER_NAME=$ECR_REPO:$BUILD_NUMBER --kubeconfig=$KUBECONFIG
                    ubuntu
                    sudo /usr/local/bin/kubectl rollout status deployment/$DEPLOYMENT_NAME --kubeconfig=$KUBECONFIG
                    ubuntu
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
