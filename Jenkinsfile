pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'Docker'
        IMAGE_NAME = 'satyasaia99/myntra'
        IMAGE_TAG = "v1.${BUILD_NUMBER}"
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/SatyasaiA99/myntrajava.git'
            }
        }

        stage('Build WAR') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKERHUB_CREDENTIALS}",
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    docker push ${IMAGE_NAME}:latest
                    docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                echo "===== VALIDATION ====="
                whoami

                if [ ! -f "$KUBECONFIG" ]; then
                  echo "❌ kubeconfig not found!"
                  exit 1
                fi

                echo "===== USING KUBECONFIG ====="
                export KUBECONFIG=$KUBECONFIG

                kubectl config use-context minikube
                kubectl get nodes

                echo "===== DEPLOY ====="
                kubectl apply -f deployment.yml
                kubectl apply -f service.yml

                echo "===== UPDATE IMAGE ====="
                kubectl set image deployment/myntra-deploy myntra-container=${IMAGE_NAME}:${IMAGE_TAG}

                echo "===== ROLLOUT ====="
                kubectl rollout status deployment/myntra-deploy --timeout=120s
                '''
            }
        }

        stage('Get Application URL') {
            steps {
                sh '''
                export KUBECONFIG=$KUBECONFIG

                kubectl get svc myntra-service

                NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[0].address}')
                NODE_PORT=$(kubectl get svc myntra-service -o jsonpath='{.spec.ports[0].nodePort}')

                echo "===================================="
                echo "Application URL: http://$NODE_IP:$NODE_PORT"
                echo "===================================="
                '''
            }
        }
    }

    post {

        success {
            emailext(
                to: 'saiankam69@gmail.com',
                subject: "SUCCESS: ${JOB_NAME} #${BUILD_NUMBER}",
                body: "Deployment Successful. Check: ${BUILD_URL}"
            )
        }

        failure {
            emailext(
                to: 'saiankam69@gmail.com',
                subject: "FAILED: ${JOB_NAME} #${BUILD_NUMBER}",
                body: "Deployment Failed. Check: ${BUILD_URL}"
            )
        }

        always {
            echo "Pipeline execution completed."
        }
    }
}
