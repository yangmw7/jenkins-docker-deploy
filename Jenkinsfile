pipeline {
    agent any

    environment {
        SERVICE_HOST = "ubuntu@172.31.8.78"
        SSH_KEY      = "/var/lib/jenkins/.ssh/jenkins_deploy"
        IMAGE_NAME   = "yangmw7/jenkins-web"
        IMAGE_TAG    = "${BUILD_NUMBER}"
    }

    stages {

        stage('Test (Quality Gate)') {
            steps {
                sh '''
                echo "Running CI Test..."
                
                # 최소 품질 게이트 예시
                # index.html 파일이 존재하지 않으면 실패
                test -f index.html

                echo "Test Passed"
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                docker build --no-cache -t ${IMAGE_NAME}:${IMAGE_TAG} .
                docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Docker Push') {
            steps {
                sh '''
                docker push ${IMAGE_NAME}:${IMAGE_TAG}
                docker push ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Deploy to Service EC2') {
            steps {
                sh '''
                ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${SERVICE_HOST} "
                    docker stop jenkins-web || true
                    docker rm jenkins-web || true
                    docker pull ${IMAGE_NAME}:latest
                    docker run -d --name jenkins-web -p 80:80 ${IMAGE_NAME}:latest
                "
                '''
            }
        }
    }
}
