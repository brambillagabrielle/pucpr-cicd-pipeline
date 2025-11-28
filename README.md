# DevOps Project: CI/CD Pipeline with Jenkins and Docker

<img src="https://img.shields.io/badge/Jenkins-49728B?style=for-the-badge&logo=jenkins&logoColor=white"> <img src="https://img.shields.io/badge/Docker-2CA5E0?style=for-the-badge&logo=docker&logoColor=white">

## About the Project

This project was developed as proposed in the **Cisco DevNet Associate - DevOps** course from the Postgraduate Program in Cloud Computing Services and Systems Engineering (PUCPR). The objectives of this work were to practice theoretical concepts learned during the course by implementing a CI/CD Pipeline using Jenkins and Docker, with the following stages:
1. Cleanup
2. Checkout
3. Build
4. Delivery

### Project Structure

```
├── Jenkinsfile
├── db
    ├── Dockerfile
    └── codigo.sql
└── web
    ├── Dockerfile
    ├── main.py
    └── requirements.txt
```

## Prerequisites

### Local Jenkins Setup

To perform the project activities, it was necessary to prepare the required tools by **setting up Jenkins in the local environment**, enabling the creation and execution of the developed pipeline.

For this, the following command was used in the Linux WSL terminal, allowing **Docker build and other operations within the Jenkins container**:

```
docker run -d --name jenkins -u root -p 8080:8080 -p 50000:50000 -v jenkins-data:/var/jenkins_home -v $(which docker):/usr/bin/docker -v /var/run/docker.sock:/var/run/docker.sock jenkins/jenkins:lts-jdk17
```

To validate the setup and retrieve the initial administrator user credentials, the commands were monitored with: `docker logs -f jenkins`

After the container successfully started, Jenkins could be accessed through the browser at `http://localhost:8080`

### Docker Hub Repository and Access Token

The next tool involved in the activity is [Docker Hub](hub.docker.com/), for **pushing the images** that will be built in the final stages of the pipeline. 

For this purpose, it was necessary to create **two repositories for each of the images (db and web)**, as well as an access token, which was added as an **access credential in Jenkins** with "Read, Write, Delete" permissions.

## Pipeline Structure

### 1. Cleanup

The first stage, called "Cleanup", corresponds to **cleaning the Workspace (Jenkins working directory)** using the **Workspace Cleanup plugin**:

```
stage('Cleanup') {
    steps {
        echo "Cleaning the workspace..."
        cleanWs()
    }
}
```

### 2. Checkout

Next, "Checkout" performs a **clone of a specified Git repository** with the content from a specific branch, using the Git plugin:

```
stage('Checkout') {
    steps {
        echo "Performing Git repository checkout..."
        git branch: 'master', url: 'https://github.com/brambillagabrielle/pucpr-devops-project'
    }
}
```

### 3. Build

After that, with the repository content in the Jenkins working directory, the "Build" step performs the **construction of both images with Docker**.

As indicated by the project structure, each of the 2 directories are specified, which contain **Dockerfiles indicating the step-by-step process for packaging the application and the DB**, and passing a "name:tag" based on the **Docker Hub repository and the Jenkins build number**:

```
stage('Build') {
    steps {
        echo "Building Docker images..."
        sh "docker build -t brambillagabi/imagem-db:${BUILD_NUMBER} ./db"
        sh "docker build -t brambillagabi/imagem-web:${BUILD_NUMBER} ./web"
    }
}
```

### 4. Delivery

Finally, after the images are ready, the last stage, "Delivery", is used to **send the images to the Docker Hub repositories**, performing login with the credentials created and registered for this purpose:

```
stage('Delivery') {
    steps {
        echo "Delivering Docker images..."
        withCredentials([usernamePassword(credentialsId: 'docker-token',
                usernameVariable: 'DOCKER_USER',
                passwordVariable: 'DOCKER_PASS')]) {
            sh '''echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'''
            sh "docker push brambillagabi/imagem-db:${BUILD_NUMBER}"
            sh "docker push brambillagabi/imagem-web:${BUILD_NUMBER}"
        }
    }
}
```
