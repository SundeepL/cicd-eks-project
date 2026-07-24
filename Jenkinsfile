pipeline {
    agent any

    environment {
        IMAGE_NAME = "sundeep04/portfolio"
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push $IMAGE_NAME:$IMAGE_TAG'
            }
        }

        stage('Deploy to Amazon EKS') {
            steps {
                sh '''
                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml
                    kubectl rollout restart deployment portfolio-deployment
                    kubectl rollout status deployment portfolio-deployment
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully! Application deployed to Amazon EKS.'
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}
