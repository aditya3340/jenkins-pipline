pipeline {
    agent any

    environment {
        KUBECONFIG = "/var/jenkins_home/.kube/config" // Optional, if needed
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Checkout code from the repository
                checkout scm
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    // Use kubectl from mounted path
                    sh '/kubectl-bin/kubectl apply -f nginx-deployment.yaml'
                    sh '/kubectl-bin/kubectl apply -f nginx-service.yaml'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    // Use kubectl from mounted path again to verify
                    sh '/kubectl-bin/kubectl get pods -n app'
                    sh '/kubectl-bin/kubectl port-forward svc/nginx -n app 80:80'
                }
            }
        }
    }
}
