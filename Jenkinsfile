pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "mo4222/my-nginx-app:latest"
        DOCKER_CONTAINER = "nginx-container"
        EC2_USER = "ubuntu" 
        EC2_HOST = "13.217.201.72"
    }

    stages {


        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker build -t $DOCKER_IMAGE .
                        docker push $DOCKER_IMAGE
                    """
                }
            }
        }

        stage('Deploy on EC2') {
            steps {
                withCredentials([
                    sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'EC2_KEY')
                ]) {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            ssh -o StrictHostKeyChecking=no -i $EC2_KEY $EC2_USER@$EC2_HOST '
                                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin &&
                                docker stop $DOCKER_CONTAINER || true &&
                                docker rm $DOCKER_CONTAINER || true &&
                                docker pull $DOCKER_IMAGE &&
                                docker run -d --name $DOCKER_CONTAINER -p 81:81 $DOCKER_IMAGE
                            '
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Deployment successful!"
        }
        failure {
            echo "Deployment failed. Check logs!"
        }
    }
}
