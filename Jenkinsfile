pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'Docker'          // Jenkins DockerHub credentials ID
        IMAGE_NAME = 'satyasaia99/myntra'
        EMAIL_RECIPIENT = 'saiankam69@gmail.com'  // Your email for notifications
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
                sh "docker build -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${IMAGE_NAME}:latest
                        docker logout
                    '''
                }
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                    docker stop myntra || true
                    docker rm myntra || true
                    docker run -d -p 5656:8080 --name myntra ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        kubectl apply -f deployment.yml
                        kubectl apply -f service.yml
                    '''
                }
            }
        }
    }

    post {
        success {
            emailext(
                to: "${EMAIL_RECIPIENT}",
                subject: "✅ Jenkins Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """The Jenkins build ${env.JOB_NAME} #${env.BUILD_NUMBER} was successful.
Check details at: ${env.BUILD_URL}"""
            )
        }
        failure {
            emailext(
                to: "${EMAIL_RECIPIENT}",
                subject: "❌ Jenkins Build FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """The Jenkins build ${env.JOB_NAME} #${env.BUILD_NUMBER} failed.
Check details at: ${env.BUILD_URL}"""
            )
        }
    }
}
