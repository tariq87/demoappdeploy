pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        APP_NAME = "python-flask-app"
        IMAGE_NAME = "${DOCKERHUB_CREDENTIALS_USR}/${APP_NAME}"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Test') {
            steps {
                script {
                    docker.image('python:3.9-slim').inside {
                        sh 'pip install -r requirements.txt'
                        sh 'python -m unittest test_app.py'
                    }
                }
            }
        }
        
        stage('Build and Push Docker Image') {
            steps {
                script {
                    def dockerImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                    docker.withRegistry('', DOCKERHUB_CREDENTIALS) {
                        dockerImage.push()
                        dockerImage.push('latest')
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    docker.image('bitnami/kubectl:latest').inside {
                        withKubeConfig([credentialsId: 'kubernetes-config']) {
                            sh "sed -i 's|{{IMAGE_NAME}}|${IMAGE_NAME}:${IMAGE_TAG}|' k8s-deployment.yaml"
                            sh "kubectl apply -f k8s-deployment.yaml"
                        }
                    }
                }
            }
        }
    }
}
