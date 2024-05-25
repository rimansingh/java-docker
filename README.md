# Setting Up CI/CD Pipeline with Vagrant, Docker, and Jenkins

## Description
Git Checkout: Clones the repository from GitHub.
Compile: Compiles the Java code using Maven.
SonarQube Analysis: Runs SonarQube scanner for code quality analysis.
Trivy Scan: Scans the project for vulnerabilities using Trivy.
Code Build: Builds the project using Maven.
Docker Build & Push: Builds the Docker image and pushes it to Docker Hub.
Docker Deploy: Deploys the Docker container from the Docker Hub image.

## Prerequisites
Jenkins server with required plugins: Maven Integration, Git, Docker, SonarQube Scanner.
SonarQube server setup.
Docker installed on the Jenkins server.
Trivy installed on the Jenkins server.
Docker Hub account with credentials configured in Jenkins.

## 1. Install Vagrant
Vagrant simplifies the setup of development environments. Follow these steps to install Vagrant:

Download the appropriate installer from Vagrant Downloads.

Install Vagrant by running the installer.
OR
Create a Vagrantfile and paste
```
Vagrant.configure("2") do |config|

  ### Master VM ####
  config.vm.define "cicd" do |cicd|
    cicd.vm.box = "ubuntu/jammy64"
    cicd.vm.hostname = "cicd"
    cicd.vm.network "private_network", ip: "192.168.56.20"
    cicd.vm.provider "virtualbox" do |vb|
      vb.memory = "3072"
      vb.cpus = 2
    end
  end
  
end
```
then
```
vagrant up
vagrant ssh
```

## 2. Install Docker
Docker allows you to package and run applications in containers. Here's how to install Docker:
```
sudo apt-get update -y
sudo apt install -y docker.io
```

## 3. Install Jenkins
Jenkins is a popular automation server. Here's how to install Jenkins:

Install JDK
```
sudo apt update
sudo apt install fontconfig openjdk-17-jre
```
Install Jenkins
```
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

## 4. Configure Jenkins Plugins
Jenkins requires plugins to extend its functionality. Follow these steps to install necessary plugins:

Navigate to Jenkins Dashboard -> Manage Jenkins -> Plugins.
Go to the "Available" tab and search for the following plugins:
jdk
sonar scanner
docker
Select the plugins and click "Install without restart".

## 5. Set Up CI/CD Pipeline
Now, let's configure Jenkins to set up a CI/CD pipeline:
The following Jenkins pipeline code outlines the steps for continuous integration and deployment:
```
pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3.8'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/rimansingh/java-docker.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                sh ''' 
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.url=http://192.168.56.20:9000 \
                        -Dsonar.login=squ_b6b0c525ef9570b2usoef945c71e2e3ce21e \
                        -Dsonar.projectName=docker-desktop \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=docker-desktop \
                        -X
                    '''
            }
        }
        
        stage('Trivy Scan') {
            steps {
                sh " trivy fs --security-checks vuln, config /var/lib/jenkins/workspace/docker-desktop"
            }
        }
        
        stage('Code Build') {
            steps {
                sh "mvn clean install"
            }
        }
        
        stage('Docker Build & Push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'your-docker-username-and-password'){
                        sh "docker build -t docker-desktop ."
                        sh "docker tag docker-desktop rimandeepsingh/docker-desktop:latest"
                        sh "docker push rimandeepsingh/docker-desktop:latest"
                    }
                }
            }
        }
        
        stage('Docker Deploy') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'your-docker-username-and-password'){
                        sh "docker run -d --name docker-desk -p 8081:8081 rimandeepsingh/docker-desktop:latest"
                    }
                }
            }
        }
    }
}

```





