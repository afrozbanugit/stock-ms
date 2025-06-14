pipeline {
    agent any
    environment{
	APP_DIR='/var/lib/jenkins/workspace/stock-ms/'
	JAR_FILE='stock-ms-0.0.1-SNAPSHOT.jar'
    IMAGE = 'aafrozbanu/stock-ms'
    IMAGE_TAG = "build-${BUILD_NUMBER}"
    DOCKER_CREDENTIALS_ID = 'docker-registry-login-id'
    AWS_CREDENTIALS_ID='Aws-login-id'
    AWS_REGION='us-west-1'
    CLUSTER_NAME='proj1-eks-cluster'
	}
    stages {
         stage('Clean workspace'){
            steps{
               cleanWs() 
            }
        }
        stage('Cloning Git Repo') {
            steps {
                script{
                    sh 'git clone "https://github.com/afrozbanugit/stock-ms.git"'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {                    
                    sh 'docker build -t ${IMAGE}:${IMAGE_TAG} stock-ms/.'
                    sh "docker tag ${IMAGE}:${IMAGE_TAG} ${IMAGE}:latest"
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {                    
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        docker.image("${IMAGE}:${IMAGE_TAG}").push()
                        docker.image("${IMAGE}:latest").push()
                    }
                }
            }
        }
        stage('Deploy to EKS') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: "${AWS_CREDENTIALS_ID}"
                ]]) {
                    sh '''
                        aws eks --region $AWS_REGION update-kubeconfig --name $CLUSTER_NAME
                        sh "sed -i 's|image: .*|image: ${IMAGE}:${IMAGE_TAG}|' stock-ms/k8s-manifests/deployment.yml"
                        sh "kubectl apply -f stock-ms/k8s-manifests/deployment.yml"
                        kubectl apply -f stock-ms/k8s-manifests/service.yml
                    '''
                }
            }
        }
    }
    post {
        success {
            echo "✅ Deployed ${IMAGE}:${IMAGE_TAG} to EKS"
        }
        failure {
            echo "❌ Deployment failed."
        }
    }
}
