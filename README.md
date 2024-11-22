# DevOps Project - The Ultimate CI/CD Corporate DevOps Pipeline Project with Jenkins, SonarQube, Nexus and Kubernetes deployment

## Introduction
In today's fast-paced development environment, continuous integration and continuous delivery (CI/CD) are essential practices for delivering high-quality software rapidly and reliably. This project walks through setting up a robust CI/CD pipeline from scratch, utilizing industry-standard tools like **Jenkins**, **Docker**, **Trivy**, **SonarQube**, and **Nexus**. 

This guide is designed for developers and DevOps enthusiasts who want to automate the process of building, testing, and deploying applications in a scalable and secure manner.

We will:
- Set up the necessary infrastructure on AWS.
- Install and configure Docker, Jenkins, Trivy, Nexus, and SonarQube.
- Create a Jenkins pipeline to automate CI/CD, ensuring continuous builds, vulnerability scanning, code quality analysis, and deployment with minimal manual intervention.

Let’s dive in and transform your software delivery process with this powerful CI/CD setup.

---

## PHASE 1: INFRASTRUCTURE SETUP 🛠️

### 1. Creating 3 Ubuntu 24.04 VM Instances on AWS 🌐
#### Steps:
1. **Sign in to AWS Management Console:**
   - Go to the [AWS Management Console](https://aws.amazon.com/console/).
   - Sign in with your AWS account credentials.

2. **Navigate to EC2 dashboard & Launch Instances:**
   - In the AWS Management Console, search for **EC2**.
   - Click **Launch Instance** and follow these steps:
     - Choose **Ubuntu Server 24.04 LTS** as the AMI.
     - Select an instance type (e.g., `t2.micro`).
     - Configure security group: Allow SSH (port 22) from your IP.
     - Launch the instance with an appropriate key pair.

3. **Access Your Instances:**
   - Use an SSH client like **Git Bash**:
     ```sh
     ssh -i /path/to/key.pem ubuntu@your-instance-ip
     ```

---

### 2. Install Docker on All 3 VMs 🐳
#### Steps:
1. Update packages:
   ```sh
   sudo apt-get update
   sudo apt-get install -y ca-certificates curl
   ```

2. Add Docker GPG key:
   ```sh
   sudo install -m 0755 -d /etc/apt/keyrings
   sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
   sudo chmod a+r /etc/apt/keyrings/docker.asc
   ```

3. Add Docker repository:
   ```sh
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

4. Update package index & Install Docker:
   ```sh
   sudo apt-get update
   sudo apt-get install -y docker-ce docker-ce-cli containerd.io -y
   ```

5. Add user to docker group:
   ```sh
   sudo usermod -aG docker $USER
   newgrp docker
   ```

---

### 3. Setting Up Jenkins on Ubuntu 🔧
#### Steps:
1. Install Java:
   ```sh
   sudo apt install -y fontconfig openjdk-17-jre
   ```

2. Add Jenkins repository:
   ```sh
   sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
   echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
   ```

3. Install Jenkins:
   ```sh
   sudo apt-get update
   sudo apt-get install -y jenkins
   ```

4. Start Jenkins:
   ```sh
   sudo systemctl start jenkins
   sudo systemctl enable jenkins
   ```

5. Access Jenkins:
   - Open http://your-server-ip:8080 in a browser.
   - Retrieve the initial admin password:
   ```sh
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```
---

### 4. Installing Trivy on Jenkins Server 🔍
#### Steps:
1. Install prerequisites:
   ```sh
   sudo apt-get install -y wget apt-transport-https gnupg lsb-release
   ```

2. Add Trivy repository:
   ```sh
   wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
   echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/trivy.list
   ```

3. Install Trivy:
   ```sh
   sudo apt-get update
   sudo apt-get install -y trivy
   ```

---

### 5. Setting Up Nexus Repository Manager Using Docker 📦
#### Steps:
1. Pull the Nexus Docker image:
   ```sh
   sudo docker pull sonatype/nexus3
   ```

2. Run the Nexus container:
   ```sh
   sudo docker run -d -p 8081:8081 --name nexus -v nexus-data:/nexus-data sonatype/nexus3
   ```

3. Access Nexus:
   - Navigate to http://your-server-ip:8081 in a browser.
   - Retrieve the default admin password
   ```sh
   sudo docker logs nexus 2>&1 | grep -i password
   ```

**Alternate Way:**

4. Run Nexus container:
   ```sh
   sudo docker run -d --name nexus -p 8081:8081 sonatype/nexus3
   ```

---

### 6. Setting Up SonarQube Using Docker 📈
#### Steps:
1. Create a network:
   ```sh
   sudo docker network create sonarnet
   ```

2. Run PostgreSQL container:
   ```sh
   sudo docker run -d --name sonarqube_db --network sonarnet -e POSTGRES_USER=sonar -e POSTGRES_PASSWORD=sonar -e POSTGRES_DB=sonarqube postgres:latest
   ```

3. Run SonarQube container:
   ```sh
   sudo docker run -d --name sonarqube --network sonarnet -p 9000:9000 -e sonar.jdbc.url=jdbc:postgresql://sonarqube_db:5432/sonarqube -e sonar.jdbc.username=sonar -e sonar.jdbc.password=sonar sonarqube:latest
   ```

4. Access SonarQube:
   - Navigate to http://your-server-ip:9000.
   - Login with admin as username and password.

**Alternate Way:**

5. Run SonarQube container:
   ```sh
   sudo docker run -d --name sonarqube -p 9000:9000 sonarqube:lts-community
   ```

---


## PHASE 2: JENKINS PIPELINE AUTOMATION 🚀

### 1. **Sign in to Jenkins:**
   - In Jenkins, click on "New Item" and select "Pipeline".
   - Name the pipeline and click "OK".

### 2. **Configure the Pipeline:**
   - Scroll down to the "Pipeline" section.
   - Select "Pipeline script" and define your pipeline stages using Groovy.

### 3. Sample Jenkins Pipeline Script:

```sh
pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/your-repo.git'
            }
        }
        stage('Build with Maven') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Docker Build and Push') {
            steps {
                script {
                    docker.build("your-app:latest").push("your-docker-repo/your-app:latest")
                }
            }
        }
        stage('Security Scan with Trivy') {
            steps {
                sh 'trivy image your-docker-repo/your-app:latest'
            }
        }
        stage('Quality Analysis with SonarQube') {
            steps {
                withSonarQubeEnv('SonarQube Server') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
    }
}
```

### 4. **Save and Run the Pipeline:**
   - Save the pipeline configuration.
   - Click "Build Now" to run the pipeline.

This pipeline will automate the entire CI/CD process, from cloning the repository to building the application, scanning it for vulnerabilities with Trivy, and analyzing code quality with SonarQube.