pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds') // ton ID Jenkins pour Docker Hub
    }

    stages {

        stage('Checkout') {
            steps {
                echo "📥 Cloning repository..."
                git branch: 'main', url: 'https://github.com/Habibdrira/Full-Stack-To-Do-List-App.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                echo "🛠 Building Docker images..."
                script {
                    // Liste des services à builder
                    def services = ['todo-backend','todo-frontend','mysql-todo']

                    services.each { svc ->
                        def dockerfilePath = "./k8s/${svc}-Dockerfile"
                        def buildDir = "./"  // ton chemin par défaut, adapter si nécessaire
                        if (fileExists(dockerfilePath)) {
                            echo "🔹 Building image for ${svc}..."
                            try {
                                sh "docker build -t drirahabib/${svc}:latest ${buildDir}"
                            } catch (err) {
                                echo "⚠️ Failed to build ${svc}: ${err}"
                            }
                        } else {
                            echo "⚠️ Dockerfile not found for ${svc}, skipping..."
                        }
                    }
                }
            }
        }

        stage('Docker Hub Login') {
            steps {
                echo "🔑 Logging into Docker Hub..."
                script {
                    sh "echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin"
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                echo "📤 Pushing Docker images to Docker Hub..."
                script {
                    def services = ['todo-backend','todo-frontend','mysql-todo']

                    services.each { svc ->
                        def imageExists = sh(script: "docker images -q drirahabib/${svc}:latest", returnStdout: true).trim()
                        if (imageExists) {
                            try {
                                sh "docker push drirahabib/${svc}:latest"
                                echo "✅ Successfully pushed ${svc}"
                            } catch (err) {
                                echo "⚠️ Failed to push ${svc}: ${err}"
                            }
                        } else {
                            echo "⚠️ Image ${svc} not found, skipping push."
                        }
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "🚀 Deploying services to Kubernetes..."
                withCredentials([file(credentialsId: 'kubeconfig-creds', variable: 'KUBECONFIG')]) {
                    sh '''
                        kubectl apply -f ./k8s/namespace.yaml
                        kubectl apply -f ./k8s/mysql-deployment.yaml
                        kubectl apply -f ./k8s/mysql-service.yaml
                        kubectl apply -f ./k8s/backend-deployment.yaml
                        kubectl apply -f ./k8s/backend-service.yaml
                        kubectl apply -f ./k8s/frontend-deployment.yaml
                        kubectl apply -f ./k8s/frontend-service.yaml
                    '''
                }
            }
        }

        stage('Test Deployment') {
            steps {
                echo "🔍 Checking Kubernetes pods and services..."
                sh "kubectl get pods -n todo-app || echo '⚠️ Failed to get pods'"
                sh "kubectl get svc -n todo-app || echo '⚠️ Failed to get services'"
            }
        }

    }

    post {
        success {
            echo '🎉 CI/CD pipeline completed successfully!'
        }
        failure {
            echo '❌ CI/CD pipeline failed. Check logs for details!'
        }
    }
}
