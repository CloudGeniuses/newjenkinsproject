pipeline {
    tools {
        maven 'maven'
    }
    agent any
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/soma2907/newjenkinsproject.git']]])
            }
        }
        stage('Build Jar') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Docker Image Build') {
            steps {
                sh 'docker build -t cloudgenius .'
            }
        }
        stage('Push Docker Image to ECR') {
            steps {
                withAWS(credentials: 'aws-cred', region: 'us-east-1') {
                    sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 211125403425.dkr.ecr.us-east-1.amazonaws.com'
                    sh 'docker tag cloudgenius:latest 211125403425.dkr.ecr.us-east-1.amazonaws.com/cloudgenius:latest'
                    sh 'docker push 211125403425.dkr.ecr.us-east-1.amazonaws.com/cloudgenius:latest'
                }
            }
        }
        stage('Integrate Jenkins with EKS Cluster and Deploy App') {
            steps {
                withAWS(credentials: 'aws-credentials', region: 'us-east-1') {
                  script {
                    sh ('aws eks --region us-east-1 update-kubeconfig --name cloudgeniusk8s')
                    sh '/var/lib/jenkins/kubectl apply -f eks-deploy-k8s.yaml'
                }
                }
        }
    }
    }
}
