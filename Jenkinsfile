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

                    //create namespace
                    sh '''
                          if ! /kubectl-bin/kubectl get namespace app > /dev/null 2>&1; then
                            /kubectl-bin/kubectl create namespace app
                          else
                            echo "Namespace 'app' already exists."
                          fi
                        '''

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

                    // wait loop
                    sh '''
                      echo "Waiting for nginx pod to be ready..."
                      for i in {1..30}; do
                        READY=$(kubectl get pods -n app -l app=nginx -o jsonpath="{.items[0].status.containerStatuses[0].ready}")
                        STATUS=$(kubectl get pods -n app -l app=nginx -o jsonpath="{.items[0].status.phase}")
                        if [ "$READY" == "true" ] && [ "$STATUS" == "Running" ]; then
                          echo "Pod is ready!"
                          break
                        fi
                        echo "Pod not ready yet... retrying in 5s"
                        sleep 5
                      done
                      '''
                    
                    sh '/kubectl-bin/kubectl port-forward svc/nginx -n app 80:80'
                }
            }
        }
    }
}
