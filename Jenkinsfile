pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        DOCKERHUB_USER = "mo4222"
        IMAGE_NAME = "my-nginx-app"
        IMAGE_TAG = "v1"
        EC2_USER = "my-nginx-app"  
        EC2_HOST = "13.217.201.72" 
        PEM_KEY_PATH = "/var/lib/jenkins/my-nginx-app.pem"
    }

    stages {
        stage('Check Docker Installed') {
            steps {
                sh 'docker --version'
                sh 'docker ps'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Login to DockerHub') {
            steps {
                sh """
                echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh """
                ssh -o StrictHostKeyChecking=no -i ${PEM_KEY_PATH} ${EC2_USER}@${EC2_HOST} '
                  docker login -u ${DOCKERHUB_CREDENTIALS_USR} -p ${DOCKERHUB_CREDENTIALS_PSW} &&
                  docker pull ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} &&
                  docker rm -f nginx-container || true &&
                  docker run -d -p 8080:80 --name nginx-container ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                '
                """
            }
        }
    }
}
