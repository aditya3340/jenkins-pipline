pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Deploy to Minikube') {
            steps {
                // Deploy your Kubernetes YAML manifests
                sh '/tmp/kubectl apply -f nginx-deployment.yaml'
                sh '/tmp/kubectl apply -f nginx-service.yaml'
            }
        }
    }
}
