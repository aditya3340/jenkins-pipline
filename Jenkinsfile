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
                sh 'kubectl apply -f manifests/deployment.yaml'
                sh 'kubectl apply -f manifests/service.yaml'
            }
        }
    }
}
