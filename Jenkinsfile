pipeline {
    agent any

    environment {
        BACKEND_IMAGE = "drirahabib/todo-backend:latest"
        FRONTEND_IMAGE = "drirahabib/todo-frontend:latest"
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials') // ID de vos credentials Docker Hub
        KUBECONFIG = credentials('kubeconfig-credentials') // ID de vos credentials Kubeconfig
    }

    stages {

        stage('Checkout SCM') {
            steps {
                echo "📥 Cloning repository..."
                checkout scm
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    // Backend
                    if (fileExists('./To-Do-List-App-SpringBoot/pom.xml')) {
                        echo "🛠 Building backend Docker image..."
                        sh "cd To-Do-List-App-SpringBoot && ./mvnw clean package -DskipTests"
                        sh "docker build -t ${BACKEND_IMAGE} ./To-Do-List-App-SpringBoot"
                    } else {
                        echo "⚠️ Backend not found, skipping..."
                    }

                    // Frontend
                    if (fileExists('./To-Do-List-App-Angular/package.json')) {
                        echo "🛠 Building frontend Docker image..."
                        sh "docker build -t ${FRONTEND_IMAGE} ./To-Do-List-App-Angular"
                    } else {
                        echo "⚠️ Frontend not found, skipping..."
                    }

                    // MySQL
                    if (!fileExists('./k8s/mysql-deployment.yaml')) {
                        echo "⚠️ Pas de Dockerfile pour MySQL, skipping..."
                    }
                }
            }
        }

        stage('Docker Hub Login & Push') {
            steps {
                script {
                    echo "🔑 Logging into Docker Hub..."
                    sh """
                        echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin
                    """

                    // Push images
                    sh "docker push ${BACKEND_IMAGE}"
                    sh "docker push ${FRONTEND_IMAGE}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "🚀 Deploying services to Kubernetes..."
                    sh "kubectl apply -f ./k8s --kubeconfig=${KUBECONFIG}"
                }
            }
        }

        stage('Test Deployment') {
            steps {
                script {
                    echo "🔍 Checking Kubernetes pods and services..."
                    sh "kubectl get pods -n default --kubeconfig=${KUBECONFIG}"
                    sh "kubectl get svc -n default --kubeconfig=${KUBECONFIG}"
                }
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
            node {
                cleanWs()
            }
        }
    }
}
