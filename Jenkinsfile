pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = "abhi2404"
        IMAGE_NAME = "devops-project5"
        IMAGE_TAG = "latest"
        FULL_IMAGE = "${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/abhi-gadge1773/End-to-End-CI-CD-with-Docker-Jenkins-Ansible.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${FULL_IMAGE}"
                    sh "docker build -t ${FULL_IMAGE} ."
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    script {
                        echo "Logging in and pushing ${FULL_IMAGE} to Docker Hub"
                        sh '''
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker push "$DOCKER_USER/$IMAGE_NAME:$IMAGE_TAG"
                        '''
                    }
                }
            }
        }

        stage('Run Ansible Deployment') {
            steps {
                script {
                    echo "Running Ansible Playbook for deployment"
                    sh '''
                        export LC_ALL=en_US.UTF-8
                        export LANG=en_US.UTF-8
                        ansible-playbook -i inventory.ini ansible/deploy.yml
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs above for details."
        }
        always {
            cleanWs()
        }
    }
}

