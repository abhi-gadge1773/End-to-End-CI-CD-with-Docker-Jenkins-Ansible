pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = "abhi2404"
	IMAGE_NAME = "devops-project5"
        IMAGE_TAG = "latest"
    }

    stages {
        stage('Clone') {
            steps {
                git 'https://github.com/abhi-gadge1773/End-to-End-CI-CD-with-Docker-Jenkins-Ansible.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $DOCKER_HUB_USER/$IMAGE_NAME:$IMAGE_TAG ."
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                script {
                    sh "docker push $DOCKER_HUB_USER/$IMAGE_NAME:$IMAGE_TAG"
                }
            }
        }

        stage('Run Ansible') {
            steps {
                script {
                    sh '''
                        export LC_ALL=en_US.UTF-8
                        export LANG=en_US.UTF-8
                        ansible-playbook -i inventory.ini ansible/deploy.yml
                    '''
                }
            }
        }
    }
}

