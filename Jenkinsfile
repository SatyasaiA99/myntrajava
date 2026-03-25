pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'Docker'
        AWS_CREDENTIALS = 'aws-creds'
        IMAGE_NAME = 'satyasaia99/myntra'
        REGION = 'us-east-1'
        CLUSTER_NAME = 'mycluster20'
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

        stage('Deploy to Kubernetes (EKS)') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: "${AWS_CREDENTIALS}"
                ]]) {
                    sh '''
                        aws eks update-kubeconfig --region ${REGION} --name ${CLUSTER_NAME}
                        kubectl apply -f deployment.yml
                        kubectl apply -f service.yml
                        kubectl set image deployment/myntra-deploy myntra-container=${IMAGE_NAME}:${IMAGE_TAG}
                        kubectl rollout status deployment/myntra-deploy
                    '''
                }
            }
        }
    }

    post {
        always {
            emailext(
                to: 'mr.siddu1432@gmail.com',
                subject: "Build ${currentBuild.currentResult}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Build Status: ${currentBuild.currentResult}

Project: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Docker Image: ${IMAGE_NAME}:${IMAGE_TAG}

Cluster: ${CLUSTER_NAME}
Region: ${REGION}

Check Details: ${env.BUILD_URL}
"""
            )
        }
    }
}
