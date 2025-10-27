pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
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
                    def services = [
                        'todo-backend', 'todo-frontend', 'mysql-todo'
                    ]
                    
                    services.each { svc ->
                        def dockerfilePath = "./To-Do-List-App-SpringBoot/Dockerfile"
                        if (svc == 'todo-frontend') {
                            dockerfilePath = "./To-Do-List-App-Angular/Dockerfile"
                        } else if (svc == 'mysql-todo') {
                            dockerfilePath = "./k8s/mysql-deployment.yaml" // juste placeholder, tu n’as pas de Dockerfile pour MySQL
                        }
                        
                        if (fileExists(dockerfilePath)) {
                            echo "Building image for ${svc}..."
                            try {
                                if (svc != 'mysql-todo') {
                                    def buildPath = svc == 'todo-backend' ? './To-Do-List-App-SpringBoot' : './To-Do-List-App-Angular'
                                    sh "docker build -t drirahabib/${svc}:latest ${buildPath}"
                                } else {
                                    echo "⚠️ Pas de Dockerfile pour ${svc}, skipping build..."
                                }
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
                    def services = ['todo-backend', 'todo-frontend']
                    
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
                    sh "kubectl apply -f ./k8s"
                }
            }
        }

        stage('Test Deployment') {
            steps {
                echo "🔍 Checking Kubernetes pods and services..."
                sh "kubectl get pods -n default || echo '⚠️ Failed to get pods'"
                sh "kubectl get svc -n default || echo '⚠️ Failed to get services'"
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
