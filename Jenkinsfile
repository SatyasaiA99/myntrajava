pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'Docker'   // Jenkins DockerHub credentials ID
        AWS_CREDENTIALS = 'aws-creds'      // Jenkins AWS credentials ID
        IMAGE_NAME = 'satyasaia99/myntra'
        REGION = 'us-east-1'
        CLUSTER_NAME = 'mycluster'         // 🔁 change to your EKS cluster name
        IMAGE_TAG = "v1.${BUILD_NUMBER}"
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
                sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
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

        stage('Run Container (Optional Test)') {
            steps {
                sh '''
                    docker stop myntra || true
                    docker rm myntra || true
                    docker run -d -p 5656:8080 --name myntra ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Deploy to Kubernetes (EKS)') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: "${AWS_CREDENTIALS}"
                ]]) {
                    sh '''
                        # Configure kubeconfig dynamically
                        aws eks update-kubeconfig --region ${REGION} --name ${CLUSTER_NAME}

                        # Verify cluster access
                        kubectl get nodes

                        # Deploy application
                        kubectl apply -f deployment.yml
                        kubectl apply -f service.yml

                        # Update image in deployment (important for new builds)
                        kubectl set image deployment/myntra-deploy myntra-container=${IMAGE_NAME}:${IMAGE_TAG}

                        # Check rollout
                        kubectl rollout status deployment/myntra-deploy
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline executed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs."
        }
    }
}
