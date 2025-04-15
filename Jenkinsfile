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
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    // Use kubectl from mounted path again to verify
                    sh '/kubectl-bin/kubectl get pods -n jenkins'
                }
            }
        }
    }
}
