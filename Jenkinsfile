pipeline {
    agent any

    environment {
        KUBECONFIG = "/tmp/kubeconfig"  // Optional: Use if your kubeconfig is in a different path
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
                    // Optional: Ensure kubectl is configured correctly
                    sh 'kubectl config use-context minikube'  // Set Minikube context

                    // Deploy using the correct path to your deployment.yaml
                    // Assuming `deployment.yaml` is in `kubernetes/` directory
                    sh 'kubectl apply -f nginx-deployment.yaml'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    // Check that the pods are running in Minikube
                    sh 'kubectl get pods --all-namespaces'
                }
            }
        }
    }
}
