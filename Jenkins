pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'reddyk0808/latest'
        REMOTE_HOST = 'ubuntu@18.204.73.239'
    }

    stages {

        stage('Git Checkout Code') {
            steps {
                git branch: 'main',
                    credentialsId: 'f42740fa-f6d1-4cfb-a02d-dd119857bd1e',
                    url: 'https://github.com/reddyk0808/Ekart'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'docker-hub-token', variable: 'DOCKER_PAT')]) {
                    sh '''
                    echo "$DOCKER_PAT" | docker login -u reddyk0808 --password-stdin
                    docker push $DOCKER_IMAGE
                    docker logout
                    '''
                }
            }
        }

        stage('Deploy to Remote VM') {
            steps {
                sshagent(['7ceb42a1-a56c-4128-8efd-e710b0c4f1df']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no $REMOTE_HOST << EOF
                    docker pull $DOCKER_IMAGE
                    docker stop $CONTAINER_NAME || true
                    docker rm $CONTAINER_NAME || true
                    docker run -d --name $CONTAINER_NAME -p 80:80 $DOCKER_IMAGE
                    EOF
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully.'
        }
        failure {
            echo 'pipeline failed.'
        }
    }
}
