pipeline {
    agent any
    stages {
        stage('Cleanup') {
            steps {
                echo "Cleaning the workspace..."
                cleanWs()
            }
        }
        stage('Checkout') {
            steps {
                echo "Performing Git repository checkout..."
                git branch: 'master', url: 'https://github.com/brambillagabrielle/pucpr-devops-project'
            }
        }
        stage('Build') {
            steps {
                echo "Building Docker images..."
                sh "docker build -t brambillagabi/imagem-db:${BUILD_NUMBER} ./db"
                sh "docker build -t brambillagabi/imagem-web:${BUILD_NUMBER} ./web"
            }
        }
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
    }
}
