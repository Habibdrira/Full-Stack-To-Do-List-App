pipeline {
    agent any

    environment {
        BACKEND_IMAGE = "drirahabib/todo-backend:latest"
        FRONTEND_IMAGE = "drirahabib/todo-frontend:latest"
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials') // Docker Hub
        KUBECONFIG = credentials('kubeconfig-credentials')           // kubeconfig
    }

    options {
        skipDefaultCheckout(true)
        timeout(time: 60, unit: 'MINUTES')
    }

    stages {

        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Build Backend Docker Image') {
            when { expression { fileExists('./To-Do-List-App-SpringBoot/pom.xml') } }
            steps {
                node {
                    sh """
                        cd To-Do-List-App-SpringBoot
                        ./mvnw clean package -DskipTests
                        docker build -t ${BACKEND_IMAGE} .
                    """
                }
            }
        }

        stage('Build Frontend Docker Image') {
            when { expression { fileExists('./To-Do-List-App-Angular/package.json') } }
            steps {
                node {
                    sh "docker build -t ${FRONTEND_IMAGE} ./To-Do-List-App-Angular"
                }
            }
        }

        stage('Docker Hub Login & Push') {
            steps {
                node {
                    sh """
                        echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin
                        docker push ${BACKEND_IMAGE}
                        docker push ${FRONTEND_IMAGE}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                node {
                    sh "kubectl apply -f ./k8s --kubeconfig=${KUBECONFIG}"
                }
            }
        }

        stage('Test Deployment') {
            steps {
                node {
                    sh "kubectl get pods -n default --kubeconfig=${KUBECONFIG}"
                    sh "kubectl get svc -n default --kubeconfig=${KUBECONFIG}"
                }
            }
        }
    }

    post {
        success {
            node { echo "🎉 Pipeline completed successfully!" }
        }
        failure {
            node { echo "❌ Pipeline failed!" }
        }
        always {
            node { 
                echo "🧹 Cleaning workspace..."
                cleanWs()
            }
        }
    }
}
