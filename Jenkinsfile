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
        timeout(time: 60, unit: 'MINUTES') // Timeout global pour la pipeline
    }

    stages {

        stage('Checkout SCM') {
            steps {
                echo "📥 Cloning repository..."
                checkout scm
            }
        }

        stage('Build Backend Docker Image') {
            when { expression { fileExists('./To-Do-List-App-SpringBoot/pom.xml') } }
            steps {
                echo "🛠 Building backend Docker image..."
                sh """
                    cd To-Do-List-App-SpringBoot
                    ./mvnw clean package -DskipTests
                    docker build -t ${BACKEND_IMAGE} .
                """
            }
        }

        stage('Build Frontend Docker Image') {
            when { expression { fileExists('./To-Do-List-App-Angular/package.json') } }
            steps {
                echo "🛠 Building frontend Docker image..."
                sh "docker build -t ${FRONTEND_IMAGE} ./To-Do-List-App-Angular"
            }
        }

        stage('Docker Hub Login & Push') {
            steps {
                script {
                    echo "🔑 Logging into Docker Hub..."
                    sh """
                        echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin
                    """
                    sh "docker push ${BACKEND_IMAGE}"
                    sh "docker push ${FRONTEND_IMAGE}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "🚀 Deploying services to Kubernetes..."
                sh "kubectl apply -f ./k8s --kubeconfig=${KUBECONFIG}"
            }
        }

        stage('Test Deployment') {
            steps {
                echo "🔍 Checking Kubernetes pods and services..."
                sh "kubectl get pods -n default --kubeconfig=${KUBECONFIG}"
                sh "kubectl get svc -n default --kubeconfig=${KUBECONFIG}"
            }
        }
    }

    post {
        success {
            echo "🎉 CI/CD pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed! Check logs for details."
        }
        always {
            echo "🧹 Cleaning workspace..."
            cleanWs()
        }
    }
}
