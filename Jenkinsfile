pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-id') // Mets ici l’ID de tes credentials Jenkins pour DockerHub
        IMAGE_BACKEND = "drirahabib/todo-backend:latest"
        IMAGE_FRONTEND = "drirahabib/todo-frontend:latest"
        NAMESPACE = "todo-app"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/Habibdrira/Full-Stack-To-Do-List-App.git'
            }
        }

        stage('Build Backend Docker') {
            steps {
                dir('To-Do-List-App-SpringBoot') {
                    sh 'docker build -t $IMAGE_BACKEND .'
                }
            }
        }

        stage('Build Frontend Docker') {
            steps {
                dir('To-Do-List-App-Angular') {
                    sh 'docker build -t $IMAGE_FRONTEND .'
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                sh 'docker login -u $DOCKERHUB_CREDENTIALS_USR -p $DOCKERHUB_CREDENTIALS_PSW'
                sh 'docker push $IMAGE_BACKEND'
                sh 'docker push $IMAGE_FRONTEND'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/namespace.yaml'
                sh 'kubectl apply -f k8s/mysql-deployment.yaml'
                sh 'kubectl apply -f k8s/mysql-service.yaml'
                sh 'kubectl apply -f k8s/backend-deployment.yaml'
                sh 'kubectl apply -f k8s/backend-service.yaml'
                sh 'kubectl apply -f k8s/frontend-deployment.yaml'
                sh 'kubectl apply -f k8s/frontend-service.yaml'
            }
        }
    }
}
