pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        BACKEND_IMAGE = "drirahabib/todo-backend:latest"
        FRONTEND_IMAGE = "drirahabib/todo-frontend:latest"
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps() // Garde les timestamps dans les logs
    }

    stages {
        stage('Checkout SCM') {
            steps {
                echo "📥 Cloning repository..."
                git url: 'https://github.com/Habibdrira/Full-Stack-To-Do-List-App.git', branch: 'main'
            }
        }

        stage('Build Backend JAR') {
            steps {
                echo "📦 Building Spring Boot backend..."
                dir('To-Do-List-App-SpringBoot') {
                    sh './mvnw clean package -DskipTests'
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }

        stage('Build Frontend') {
            steps {
                echo "🛠 Building Angular frontend..."
                dir('To-Do-List-App-Angular') {
                    sh 'npm ci'
                    sh 'npm run build -- --prod'
                    archiveArtifacts artifacts: 'dist/**/*', fingerprint: true
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                echo "🐳 Building Docker images..."
                dir('To-Do-List-App-SpringBoot') {
                    sh "docker build -t ${BACKEND_IMAGE} ."
                }
                dir('To-Do-List-App-Angular') {
                    sh "docker build -t ${FRONTEND_IMAGE} ."
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                echo "⬆️ Pushing Docker images..."
                script {
                    docker.withRegistry('', 'dockerhub-credentials') {
                        sh "docker push ${BACKEND_IMAGE}"
                        sh "docker push ${FRONTEND_IMAGE}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "🚀 Deploying services to Kubernetes..."
                withKubeConfig([credentialsId: 'kubeconfig-creds']) {
                    sh 'kubectl apply -f ./k8s'
                    sh 'kubectl rollout status deployment/todo-backend -n todo-app'
                    sh 'kubectl rollout status deployment/todo-frontend -n todo-app'
                }
            }
        }

        stage('Test Deployment') {
            steps {
                echo "🔍 Checking pods and services..."
                sh 'kubectl get pods -n todo-app'
                sh 'kubectl get svc -n todo-app'
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
            cleanWs()
        }
    }
}
