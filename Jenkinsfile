pipeline {
    agent any

    environment {
        SERVICE_HOST = "ubuntu@172.31.8.78"
        SSH_KEY = "/var/lib/jenkins/.ssh/jenkins_deploy"
        IMAGE_NAME = "yangmw7/jenkins-web"
    }

    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'github-creds',
                    url: 'https://github.com/yangmw7/jenkins-docker-deploy.git',
                    branch: 'main'
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                docker build -t ${IMAGE_NAME}:latest .
                """
            }
        }

        stage('Docker Push') {
            steps {
                sh """
                docker login -u yangmw7 -p <DOCKER_HUB_TOKEN>
                docker push ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Deploy to Service EC2') {
            steps {
                sh """
                ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${SERVICE_HOST} '
                    docker pull ${IMAGE_NAME}:latest
                    docker stop jenkins-web || true
                    docker rm jenkins-web || true
                    docker run -d --name jenkins-web -p 8080:80 ${IMAGE_NAME}:latest
                '
                """
            }
        }
    }
}
