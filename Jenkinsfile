pipeline {
    agent any 
    environment {
        AWS_CREDENTIALS = credentials('aws-credentials') // AWS 자격 증명
        ECR_REPO = '266852548854.dkr.ecr.ap-northeast-2.amazonaws.com/test_jenkins' // ECR 레포지토리 URI
        REGION = 'ap-northeast-2' // ECR과 EKS의 리전
        KUBECONFIG_CREDENTIALS = credentials('kubeconfig-credentials') // Kubeconfig 자격 증명 (Jenkins Credentials에서 설정)
        CLUSTER_NAME = 'test-eks-cluster' // EKS 클러스터 이름
        DEPLOYMENT_NAME = 'deploybyjenkins' // Kubernetes Deployment 이름
        CONTAINER_NAME = 'flaskapp' // Kubernetes Deployment에서의 컨테이너 이름
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
        stage('Update Kubeconfig') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-credentials', variable: 'KUBECONFIG')]) {
                    // Kubeconfig 파일을 Jenkins 워크스페이스에 복사
                    sh 'cp $KUBECONFIG ~/.kube/config'
                    
                    // EKS 클러스터에 연결
                    sh 'aws eks update-kubeconfig --name $CLUSTER_NAME --region $REGION'
                }
            }
        }
        stage('Deploy to EKS') {
            steps {
                // Kubernetes Deployment 업데이트
                sh '''
                    kubectl set image deployment/$DEPLOYMENT_NAME $CONTAINER_NAME=$ECR_REPO:$BUILD_NUMBER
                    kubectl rollout status deployment/$DEPLOYMENT_NAME
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
