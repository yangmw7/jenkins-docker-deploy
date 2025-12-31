pipeline {
    agent any

    environment {
        SERVICE_HOST = "ubuntu@172.31.8.78"
        SSH_KEY = "/var/lib/jenkins/.ssh/jenkins_deploy"
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
                sh 'docker build --no-cache -t yangmw7/jenkins-web:latest .'
            }
        }

        stage('Docker Push') {
            steps {
                sh 'docker push yangmw7/jenkins-web:latest'
            }
        }

        stage('Deploy to Service EC2') {
            steps {
                sh """
                ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${SERVICE_HOST} '
                    docker stop jenkins-web || true
                    docker rm jenkins-web || true
                    docker pull yangmw7/jenkins-web:latest
                    docker run -d --name jenkins-web -p 8080:80 yangmw7/jenkins-web:latest
                '
                """
            }
        }
    }
}
