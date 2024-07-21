pipeline {
    agent {
        kubernetes {
            yaml '''
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: docker
                    image: docker:dind
                    command:
                    - cat
                    tty: true
                    privileged: true
                  - name: kubectl
                    image: bitnami/kubectl:latest
                    command:
                    - cat
                    tty: true
            '''
        }
    }
    
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
                sh 'pip install -r requirements.txt'
                sh 'python -m unittest test_app.py'
            }
        }
        
        stage('Build and Push Docker Image') {
            steps {
                container('docker') {
                    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                    sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                container('kubectl') {
                    sh "sed -i 's|{{IMAGE_NAME}}|${IMAGE_NAME}:${IMAGE_TAG}|' k8s-deployment.yaml"
                    sh "kubectl apply -f k8s-deployment.yaml"
                }
            }
        }
    }
    
    post {
        always {
            container('docker') {
                sh 'docker logout'
            }
        }
    }
}
