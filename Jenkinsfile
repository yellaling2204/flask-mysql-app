pipeline {
    agent any

    environment {
        DOCKER_HOST = 'unix:///var/run/docker.sock'
        DOCKER_TLS_VERIFY = ''
        DOCKER_CERT_PATH = ''
        DOCKER_API_VERSION = '1.43'
       credentialsId: 'dockerhub-creds'

    }

    stages {

        stage('Clone Repository') {
            steps {
                echo '📦 Cloning GitHub repository...'
                git branch: 'main', url: 'https://github.com/aathreya-sharma/flask-mysql-app.git'
            }
        }

        stage('Build') {
            steps {
                echo '🔧 Building Docker images...'
                retry(3) {
                    sh '''
                        export DOCKER_API_VERSION=$DOCKER_API_VERSION
                        echo "⏳ Running docker-compose build..."
                        docker-compose build || { 
                            echo "⚠️ Build failed, retrying in 5s..."; 
                            sleep 5; 
                            false; 
                        }
                    '''
                }
            }
        }

        stage('Test') {
            steps {
                echo '🧪 Running containers and tests...'
                sh '''
                    export DOCKER_API_VERSION=$DOCKER_API_VERSION
                    docker-compose up -d
                    echo "⏳ Waiting for services to start..."
                    sleep 15
                    curl --fail http://localhost:5001/ || (echo "❌ App test failed!" && exit 1)
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo '📤 Pushing Docker image to Docker Hub...'
                script {
                    sh '''
                        export DOCKER_API_VERSION=$DOCKER_API_VERSION
                        echo "$DOCKERHUB_CREDENTIALS_PSW" | docker login -u "$DOCKERHUB_CREDENTIALS_USR" --password-stdin
                        docker build -t $DOCKERHUB_CREDENTIALS_USR/flask-mysql-app:latest .
                        docker push $DOCKERHUB_CREDENTIALS_USR/flask-mysql-app:latest
                    '''
                }
            }
        }

        stage('Cleanup') {
            steps {
                echo '🧹 Cleaning up containers and images...'
                sh '''
                    export DOCKER_API_VERSION=$DOCKER_API_VERSION
                    docker-compose down -v || true
                    docker rmi $DOCKERHUB_CREDENTIALS_USR/flask-mysql-app:latest || true
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Build, test, and push completed successfully!'
        }
        failure {
            echo '❌ Build Failed — Check Logs!'
        }
    }
}