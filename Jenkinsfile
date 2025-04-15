pipeline {
    agent any

    environment {
        KUBECONFIG = "/var/jenkins_home/.kube/config" // Optional, if needed
        NAMESPACE = "app"
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

                    // Apply deployment if it doesn't exist, or trigger a restart if it does
                    sh '''
                        echo "Checking if namespace $NAMESPACE exists..."
                        if ! /kubectl-bin/kubectl get namespace $NAMESPACE > /dev/null 2>&1; then
                            echo "Creating namespace $NAMESPACE..."
                            /kubectl-bin/kubectl create namespace $NAMESPACE
                        else
                            echo "Namespace $NAMESPACE already exists."
                        fi
                    '''
                    
                    // Apply deployment and service (idempotent)
                    sh '''
                        echo "Applying nginx deployment..."
                        /kubectl-bin/kubectl apply -f nginx-deployment.yaml -n $NAMESPACE

                        echo "Applying nginx service..."
                        /kubectl-bin/kubectl apply -f nginx-service.yaml -n $NAMESPACE
                    '''

                    // Ensure deployment is rolled out and pods are running
                    sh '''
                        echo "Waiting for nginx deployment to complete..."
                        /kubectl-bin/kubectl rollout status deployment/nginx -n $NAMESPACE --timeout=5m
                    '''

                    // Check if the pod is running and ready
                    sh '''
                        echo "Waiting for nginx pod to be ready..."
                        for i in {1..30}; do
                            READY=$(kubectl get pods -n $NAMESPACE -l app=nginx -o jsonpath="{.items[0].status.containerStatuses[0].ready}")
                            STATUS=$(kubectl get pods -n $NAMESPACE -l app=nginx -o jsonpath="{.items[0].status.phase}")
                            if [ "$READY" == "true" ] && [ "$STATUS" == "Running" ]; then
                                echo "Pod is ready!"
                                break
                            fi
                            echo "Pod not ready yet... retrying in 5s"
                            sleep 5
                        done
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    // Verify that the deployment is working as expected
                    sh '/kubectl-bin/kubectl get pods -n $NAMESPACE'
                    
                    // Port forwarding or any other verification method
                    sh '/kubectl-bin/kubectl port-forward svc/nginx -n $NAMESPACE 8080:80'
                }
            }
        }
    }
}
