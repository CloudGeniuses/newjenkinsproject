pipeline {
    agent any
    environment {
        ECR_URI = "211125403425.dkr.ecr.us-east-1.amazonaws.com/cloudgenius"
        AWS_REGION = 'us-east-1'
        DOCKER_IMAGE = 'cloudgenius'
        EKS_CLUSTER = 'cloudgeniusk8s'
        KUBECTL_PATH = '~/bin/kubectl'
    }
    tools {
        maven 'maven'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/CloudGeniuses/newjenkinsproject.git']]])
            }
        }
        stage('Build Jar') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Docker Image Build') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }
        stage('Push Docker Image to ECR') {
            steps {
                withAWS(credentials: 'aws-cred', region: "${AWS_REGION}") {
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_URI}"
                    sh "docker tag ${DOCKER_IMAGE}:latest ${ECR_URI}:latest"
                    sh "docker push ${ECR_URI}:latest"
                }
            }
        }
        
        stage('Integrate Jenkins with EKS Cluster and Deploy App') {
            steps {
                withAWS(credentials: 'aws-cred', region: "${AWS_REGION}") {
                    script {
                        sh "aws eks --region ${AWS_REGION} update-kubeconfig --name ${EKS_CLUSTER}"
                        sh "${KUBECTL_PATH} apply -f eks-deploy-k8s.yaml"  // Use the defined path for kubectl
                    }
                }
            }
        }
        
    }
}
