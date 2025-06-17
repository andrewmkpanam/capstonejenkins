
# 🚀 CI/CD Pipeline Automation Using Jenkins on EC2

This document outlines the completed steps to set up and automate a CI/CD pipeline using Jenkins on an Amazon EC2 server. The pipeline supports Dockerized application deployment and integrates with a GitHub repository for continuous integration and delivery.

---

## 1. 🔧 Jenkins Installation on an EC2 Server

An Amazon EC2 instance running Amazon Linux 2 was provisioned and used as the Jenkins host.

### Steps Performed:

#### a. System Update and Java Installation
```bash
sudo yum update -y
sudo amazon-linux-extras install java-openjdk11 -y
```

#### b. Jenkins Repository Setup and Installation
```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo yum install jenkins -y
```

#### c. Enable and Start Jenkins
```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

#### d. Open Jenkins Port (8080) in EC2 Security Group
- Access AWS Console → EC2 → Security Groups → Inbound Rules
- Add a new rule:
  - Type: Custom TCP
  - Port: 8080
  - Source: Anywhere or My IP

#### e. Retrieve Admin Password
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

- The password was used to unlock Jenkins at `http://<EC2-PUBLIC-IP>:8080`.

---

## 2. 🐳 Docker Installation and Integration

Docker was installed and integrated with Jenkins on the same EC2 instance.

### Steps Performed:

#### a. Install Docker
```bash
sudo yum install docker -y
```

#### b. Enable and Start Docker
```bash
sudo systemctl enable docker
sudo systemctl start docker
```

#### c. Add Jenkins User to Docker Group
```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

#### d. Verify Docker Access for Jenkins
```bash
sudo su - jenkins
docker ps
```

---

## 3. 🔌 Jenkins Plugin Installation and Configuration

To enable CI/CD functionality, several essential plugins were installed.

### Steps Performed:

1. Logged into Jenkins at `http://<EC2-PUBLIC-IP>:8080`.
2. Navigated to **Manage Jenkins → Manage Plugins**.
3. Under the **Available** tab, searched and installed the following:
   - **Git plugin**
   - **Docker plugin**
   - **Docker Pipeline**
   - **Pipeline**
   - **GitHub Integration plugin**
4. Plugins were selected, then clicked **Download now and install after restart**.
5. Jenkins was restarted after plugin installation.

### Git Tool Configuration:
- Navigated to **Manage Jenkins → Global Tool Configuration**
- Added Git with the following path:
  ```bash
  /usr/bin/git
  ```

---

## 4. 🔐 Security Measures Applied

Security best practices were implemented:

- Jenkins credentials plugin was used to securely store tokens and passwords.
- Role-Based Authorization Strategy plugin enabled RBAC.
- API tokens replaced password authentication for automation.
- Jenkins CLI was disabled for non-admins.
- Security group rules in AWS limited access to specific IP addresses.

---

## 5. 🔗 Integration with Source Code Management (GitHub)

### Actions Performed:

- GitHub repository was integrated using the Git plugin and a personal access token.
- Jenkins credentials were used to authenticate securely.
- A webhook was created in the GitHub repository under **Settings → Webhooks** with the following URL:
  ```
  http://<EC2-PUBLIC-IP>:8080/github-webhook/
  ```
- Content type: `application/json`
- Triggered on `push` events.

---

## 6. ⚙️ Jenkins Freestyle Job for Web Application Build and Unit Tests

### Steps Taken:

- Created a new Freestyle job via **New Item → Freestyle Project**.
- Configured GitHub repository in the SCM section.
- Build trigger set to GitHub webhook.
- Build step added:
  ```bash
  echo "Building application..."
  # Sample unit test
  echo "Running unit tests..."
  ```
- Post-build actions logged build artifacts.

---

## 7. 🧩 Jenkins Pipeline Job for Web Application Deployment

### Jenkinsfile Used:
```groovy
pipeline {
  agent any

  environment {
    DOCKER_IMAGE = 'your-dockerhub-username/html-image'
  }

  stages {
    stage('Clone') {
      steps {
        git 'https://github.com/your/repo.git'
      }
    }
    stage('Build Docker Image') {
      steps {
        sh 'docker build -t $DOCKER_IMAGE .'
      }
    }
    stage('Run Container') {
      steps {
        sh 'docker run -d -p 8082:80 $DOCKER_IMAGE'
      }
    }
    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
          sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'
          sh 'docker push $DOCKER_IMAGE'
        }
      }
    }
  }
}
```

---

## 8. 🐳 Docker Image Management and Container Registry Integration

### Docker Image Creation in Jenkins:
- Defined in the Jenkinsfile using `docker build`.

### Running the Docker Container:
```bash
docker run -d -p 8082:80 your-dockerhub-username/html-image
```

- Web app accessed via:
  ```
  http://<EC2-PUBLIC-IP>:8082
  ```

### DockerHub Registry Setup:
- DockerHub credentials stored in Jenkins using **Manage Jenkins → Credentials**.
- Pipeline authenticated and pushed images using:
  ```bash
  docker login
  docker push your-dockerhub-username/html-image
  ```

---

## ✅ Final Outcome

Jenkins was successfully configured to automate the CI/CD process from source code management to Docker-based deployment. Freestyle and Pipeline jobs were set up, webhooks triggered automated builds, and Docker images were pushed and deployed efficiently.
