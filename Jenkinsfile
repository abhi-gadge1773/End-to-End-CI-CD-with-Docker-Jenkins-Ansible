pipeline {
    agent any

    options {
        timestamps()
        timeout(time: 30, unit: 'MINUTES')
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        IMAGE_NAME = "abhi2404/ci-cd-flask-app"
        TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Clone Repo') {
            steps {
                git credentialsId: 'github-ssh-key', url: 'git@github.com:abhi-gadge1773/End-to-End-CI-CD-with-Docker-Jenkins-Ansible.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$TAG .'
            }
        }

        stage('Login to DockerHub') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    docker push $IMAGE_NAME:$TAG
                    docker tag $IMAGE_NAME:$TAG $IMAGE_NAME:latest
                    docker push $IMAGE_NAME:latest
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Build and push successful: $IMAGE_NAME:$TAG"
        }

        failure {
            echo "❌ Build failed. Check Jenkins logs for details."
        }

        cleanup {
            script {
                echo "🧹 Cleaning up local Docker images"
                sh '''
                    docker rmi $IMAGE_NAME:$TAG || true
                    docker rmi $IMAGE_NAME:latest || true
                '''
            }
        }
    }
}

