pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'Docker' // Your Jenkins DockerHub credentials ID
        IMAGE_NAME = 'satyasaia99/myntra'
        IMAGE_TAG = "${BUILD_NUMBER}"
        KUBECONFIG = '/var/lib/jenkins/.kube/config' // Kubernetes config path on Jenkins node
    }

    stages {
        stage('Checkout') {
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
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker logout
                    '''
                }
            }
        }

        stage('Run Container Locally (Optional)') {
            steps {
                sh '''
                    docker stop myntra || true
                    docker rm myntra || true
                    docker run -d -p 5656:8080 --name myntra ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withEnv(["KUBECONFIG=${KUBECONFIG}"]) {
                    sh '''
                        # Apply deployment and service manifests
                        kubectl apply -f deployment.yml
                        kubectl apply -f service.yml

                        # Update image in deployment
                        kubectl set image deployment/myntra-deployment myntra=${IMAGE_NAME}:${IMAGE_TAG}

                        # Wait for rollout to finish
                        kubectl rollout status deployment/myntra-deployment

                        # Optional: Show service details
                        kubectl get svc myntra-service
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully! Myntra app is deployed to Kubernetes."
        }
        failure {
            echo "Pipeline failed. Check logs and fix errors."
        }
    }
}
