pipeline {
    agent any

    environment {
        // DockerHub credentials from Jenkins credentials
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')

        // EC2 details
        EC2_USER = "my-nginx-app"
        EC2_HOST = "13.217.201.72"
        PEM_KEY_PATH = "/var/lib/jenkins/my-nginx-app.pem"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh 'docker build -t mo4222/my-nginx-app:latest .'
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
                sh 'docker push mo4222/my-nginx-app:latest'
            }
        }

        stage('Deploy to EC2') {
            steps {
                echo "Deploying to EC2..."
                sh '''
                    ssh -o StrictHostKeyChecking=no -i $PEM_KEY_PATH $EC2_USER@$EC2_HOST "
                        echo 'Connected to EC2';
                        docker login -u $DOCKERHUB_CREDENTIALS_USR -p $DOCKERHUB_CREDENTIALS_PSW;
                        docker pull mo4222/my-nginx-app:latest;
                        docker stop nginx-container || true;
                        docker rm nginx-container || true;
                        docker run -d -p 8080:80 --name nginx-container mo4222/my-nginx-app:latest
                    "
                '''
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
