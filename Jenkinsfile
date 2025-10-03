pipeline {
    agent any

    environment {
        // DockerHub credentials from Jenkins credentials
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')

        // EC2 details
        EC2_USER = "ubuntu"
        EC2_HOST = "13.217.201.72"
        PEM_KEY_PATH = "/var/lib/jenkins/my-nginx-app.pem"

        // Docker image
        DOCKER_IMAGE = "mo4222/my-nginx-app:latest"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Login to DockerHub') {
            steps {
                echo "Logging into DockerHub..."
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "Pushing image to DockerHub..."
                sh 'docker push $DOCKER_IMAGE'
            }
        }

        stage('Deploy to EC2') {
            steps {
                echo "Deploying to EC2..."
                sh """
                    ssh -o StrictHostKeyChecking=no -i ${PEM_KEY_PATH} ${EC2_USER}@${EC2_HOST} '
                        echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin;
                        sudo docker pull ${DOCKER_IMAGE};
                        sudo docker stop nginx-container || true;
                        sudo docker rm nginx-container || true;
                        sudo docker run -d -p 8080:80 --name nginx-container ${DOCKER_IMAGE};
                    '
                """

            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs for errors."
        }
    }
}
