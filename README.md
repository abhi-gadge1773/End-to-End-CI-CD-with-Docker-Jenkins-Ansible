# End-to-End CI/CD Pipeline for Flask App

**Using GitHub → Jenkins → Docker → Ansible**

## 🚀 Project Overview

This project automates continuous integration and deployment of a Python Flask application using the following stack:

* **GitHub**: hosts the source code and triggers Jenkins via webhooks
* **Jenkins**: orchestrates the CI/CD pipeline
* **Docker**: containerizes the Flask application
* **Docker Hub**: image registry for built app images
* **Ansible**: automates deployment to remote servers using the Docker images
* **Ubuntu / CentOS EC2 instances** (or self-hosted Linux servers): serve as Jenkins and application hosts
---

## 📁 Repository Structure

```
.
├── app.py                     # Flask application entrypoint
├── requirements.txt          # Required Python packages
├── Dockerfile                # Builds Docker image for Flask app
├── Jenkinsfile               # Declarative Jenkins pipeline script
└── ansible/
    └── deploy.yml            # Ansible playbook to deploy Docker container
```

---

## ⚙️ Pipeline Flow

1. **Push code to GitHub** →
2. **Webhook triggers Jenkins job** →
3. Jenkins pipeline:

   * Clones the repo
   * Builds Flask app Docker image
   * Logs into Docker Hub and pushes image
   * Runs Ansible playbook to deploy/update container on target host
4. **Ansible**:

   * Connects via SSH to remote server
   * Pulls the newly pushed Docker image
   * Stops existing container (if any) and launches the new one

---

## 🧪 Prerequisites

* GitHub repository with access to push code
* Jenkins server installed (with Docker installed and Ansible available)
* Docker Hub account (or alternate registry) with credentials configured in Jenkins
* One or more remote servers with Docker installed for deployment
* SSH access from Jenkins to target server (via key-based authentication)

---

## 🚀 Setting Up

### 1. Install Tools (on Jenkins node)

Install dependencies on your CI server:

```bash
sudo apt update && sudo apt install -y openjdk-11-jdk docker.io ansible git curl
sudo systemctl enable --now docker
```

Add Jenkins to Docker group to allow Docker commands:

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

### 2. Configure GitHub Webhook

* In your GitHub repo: go to **Settings → Webhooks → Add webhook**
* Payload URL: `http://<your-jenkins-host>:8080/github-webhook/`
* Content type: `application/json`
* Select trigger: **Just the push event**

### 3. Add Jenkins Credentials

* Add Docker Hub credentials (username/password) under **Credentials**
* Set credentials ID (e.g., `dockerhub-pass`) for use in pipeline script

### 4. Create Jenkins Pipeline Job

* Define a new **Pipeline Job**
* Point to this repository’s Jenkinsfile (GitHub SCM)
* Ensure webhook trigger is selected

---

## 🧾 Jenkinsfile (Pipeline Definition)

```groovy
pipeline {
  agent any
  stages {
    stage('Clone Repo') {
      steps {
        git 'https://github.com/your-username/End-to-End-CI-CD-with-Docker-Jenkins-Ansible.git'
      }
    }
    stage('Build Docker Image') {
      steps {
        sh 'docker build -t your-dockerhub-username/flask-app ./'
      }
    }
    stage('Push to DockerHub') {
      steps {
        withCredentials([string(credentialsId: 'dockerhub-pass', variable: 'DOCKER_PASS')]) {
          sh 'echo $DOCKER_PASS | docker login -u your-dockerhub-username --password-stdin'
          sh 'docker push your-dockerhub-username/flask-app'
        }
      }
    }
    stage('Deploy with Ansible') {
      steps {
        sh 'ansible-playbook ansible/deploy.yml'
      }
    }
  }
}
```

---

## 🛠 Example Ansible Playbook (ansible/deploy.yml)

```yaml
---
- hosts: flask_hosts
  become: yes
  tasks:
    - name: Pull latest Docker image
      docker_image:
        name: your-dockerhub-username/flask-app
        source: pull

    - name: Stop existing container (if any)
      docker_container:
        name: flask_app
        state: absent

    - name: Run new container
      docker_container:
        name: flask_app
        image: your-dockerhub-username/flask-app:latest
        ports:
          - "80:5000"
        restart_policy: always
```

Ensure `flask_hosts` group is defined in your `/etc/ansible/hosts`, pointing to the deployment server(s).

---

## ✅ Validation & Testing

1. Push a change to `app.py` (e.g., update the landing message)
2. GitHub webhook triggers Jenkins pipeline
3. Jenkins builds the Docker image and pushes it to Docker Hub
4. Ansible playbook executes and updates the container on remote host
5. Visit the target host’s IP or domain on port 80 to see the updated Flask app

---

## 📈 Benefits

* Fully automated CI/CD pipeline — no manual deployment needed
* Containerized application ensures consistent environments
* Decoupled infrastructure with Ansible managing configuration
* Easily extensible to multi-server deployments

---

### 🚧 Troubleshooting Tips

* Confirm correct SSH key permissions for Jenkins and ansible hosts
* Ensure Docker daemon and Ansible connectivity are functional on target host
* Check Jenkins console output for errors in each pipeline stage

---

