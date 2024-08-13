pipeline {
    agent any 
    environment {
        AWS_CREDENTIALS = credentials('aws-credentials') // AWS 자격 증명
        ECR_REPO = '266852548854.dkr.ecr.ap-northeast-2.amazonaws.com/test_jenkins' // ECR 레포지토리 URI
        REGION = 'ap-northeast-2' // ECR과 EKS의 리전
    }
    
    stages { 

        stage('Build docker image') {
            steps {  
                sh 'docker build -t $ECR_REPO:$BUILD_NUMBER .'
            }
        }
        stage('login to ECR') {
            steps{
                sh '$(aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin 266852548854.dkr.ecr.region.amazonaws.com)' // ECR 로그인
            }
        }
        stage('Push image to ECR') {
            steps{
                sh 'docker push $ECR_REPO:$BUILD_NUMBER'
            }
        }

    }
    post {
        always {
            sh 'docker logout'
        }
    }
}
