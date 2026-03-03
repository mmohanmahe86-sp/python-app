pipeline {
    agent any

    environment {
        IMAGE_NAME = "mmohanmahe86/python-devops-app"
        DOCKER_HOST_IP = "172.31.45.32"
    }

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main', url: 'https://github.com/mmohanmahe86-sp/python-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                usernameVariable: 'DOCKER_USER',
                                passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                sh 'docker push $IMAGE_NAME'
            }
        }

        stage('Deploy to App EC2') {
            steps {
                sshagent(['jenkins-ec2-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@${DOCKER_HOST_IP} '
                    docker stop python-app || true &&
                    docker rm python-app || true &&
                    docker pull ${IMAGE_NAME} &&
                    docker run -d -p 5000:5000 --name python-app ${IMAGE_NAME}
                    '
                    """
                }
            }
        }
    }
}