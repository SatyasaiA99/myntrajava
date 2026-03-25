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

        stage('Deploy to Local Kubernetes') {
            steps {
                sh '''
                echo "===== DEBUG INFO ====="
                whoami
                echo $KUBECONFIG
                ls -l $KUBECONFIG

                echo "===== Setting kubeconfig ====="
                export KUBECONFIG=/var/lib/jenkins/.kube/config

                echo "===== Checking cluster ====="
                kubectl config get-contexts
                kubectl config use-context minikube
                kubectl get nodes

                echo "===== Deploying application ====="
                kubectl apply -f deployment.yml
                kubectl apply -f service.yml

                echo "===== Updating image ====="
                kubectl set image deployment/myntra-deploy myntra-container=${IMAGE_NAME}:${IMAGE_TAG}

                echo "===== Waiting for rollout ====="
                kubectl rollout status deployment/myntra-deploy --timeout=120s
                '''
            }
        }

        stage('Get Application URL') {
            steps {
                sh '''
                export KUBECONFIG=/var/lib/jenkins/.kube/config

                echo "===== Fetching Service Details ====="
                kubectl get svc myntra-service

                echo "===== Getting Node IP ====="
                NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[0].address}')

                echo "===== Getting NodePort ====="
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
                to: 'mr.siddu1432@gmail.com',
                subject: "✅ SUCCESS: ${JOB_NAME} #${BUILD_NUMBER}",
                body: """
                <h2>Deployment Successful</h2>

                <b>Project:</b> ${JOB_NAME} <br>
                <b>Build:</b> #${BUILD_NUMBER} <br>
                <b>Status:</b> SUCCESS ✅ <br><br>

                <b>Docker Image:</b> ${IMAGE_NAME}:${IMAGE_TAG} <br><br>

                Check Console: ${BUILD_URL}
                """
            )
        }

        failure {
            emailext(
                to: 'mr.siddu1432@gmail.com',
                subject: "❌ FAILED: ${JOB_NAME} #${BUILD_NUMBER}",
                body: """
                <h2>Deployment Failed</h2>

                <b>Project:</b> ${JOB_NAME} <br>
                <b>Build:</b> #${BUILD_NUMBER} <br>
                <b>Status:</b> FAILURE ❌ <br><br>

                Check Logs: ${BUILD_URL}
                """
            )
        }

        always {
            echo "Pipeline execution completed."
        }
    }
}
