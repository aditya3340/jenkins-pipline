pipeline {
    agent any

    environment {
        KUBECONFIG = "/var/jenkins_home/.kube/config"  // Optional, if needed
        NAMESPACE = "app"  // Define the namespace
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Check if namespace exists, create if not
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
                        /kubectl-bin/kubectl apply -f nginx-deployment.yaml -n $NAMESPACE --force

                        echo "Applying nginx service..."
                        /kubectl-bin/kubectl apply -f nginx-service.yaml -n $NAMESPACE
                    '''

                    // If the deployment exists, restart it, otherwise apply the deployment
                    sh '''
                        if /kubectl-bin/kubectl get deployment nginx -n $NAMESPACE > /dev/null 2>&1; then
                            echo "Deployment exists, restarting..."
                            /kubectl-bin/kubectl rollout restart deployment/nginx -n $NAMESPACE
                        else
                            echo "Deployment does not exist, creating..."
                            /kubectl-bin/kubectl apply -f nginx-deployment.yaml -n $NAMESPACE
                        fi
                    '''

                    // Ensure deployment is rolled out successfully
                    sh '''
                        echo "Waiting for nginx deployment to complete..."
                        /kubectl-bin/kubectl rollout status deployment/nginx -n $NAMESPACE --timeout=5m
                    '''

                    // Check if the pod is running and ready
                    sh '''
                        echo "Waiting for nginx pod to be ready..."
                        for i in {1..30}; do
                            READY=$(/kubectl-bin/kubectl get pods -n $NAMESPACE -l app=nginx -o jsonpath="{.items[0].status.containerStatuses[0].ready}")
                            STATUS=$(/kubectl-bin/kubectl get pods -n $NAMESPACE -l app=nginx -o jsonpath="{.items[0].status.phase}")
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
