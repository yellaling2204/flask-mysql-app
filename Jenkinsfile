pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/yellaling2204/flask-mysql-app'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build('yellaling2204/flask-mysql-app')
                }
            }
        }

        stage('Login to DockerHub & Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-creds') {
                        def app = docker.image('yellaling2204/flask-mysql-app')
                        app.push('latest')
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build and push successful!"
        }
        failure {
            echo "❌ Build failed — Check logs."
        }
    }
}
