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
                sh '''
                    docker run --rm -v "$PWD":/app -w /app python:3.9-slim /bin/bash -c "
                        pip install -r requirements.txt
                        python -m unittest test_app.py
                    "
                '''
            }
        }
        
        stage('Build and Push Docker Image') {
            steps {
                sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    docker push ${IMAGE_NAME}:latest
                    docker logout
                """
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    docker run --rm -v "$PWD":/app -w /app bitnami/kubectl:latest /bin/bash -c "
                        sed -i 's|{{IMAGE_NAME}}|${IMAGE_NAME}:${IMAGE_TAG}|' k8s-deployment.yaml
                        kubectl apply -f k8s-deployment.yaml
                    "
                """
            }
        }
    }
}
