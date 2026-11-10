
pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'hiran4024'
        IMAGE_REPO = 'hiran4024/kanban-dashboard'
        CONTAINER_NAME = 'kanban-dashboard'
    }

    options {
        timeout(time: 20, unit: 'MINUTES')
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out Kanban Dashboard source code'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image'
                sh '''
                    docker build \
                      -t ${IMAGE_REPO}:${BUILD_NUMBER} \
                      -t ${IMAGE_REPO}:latest .
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DH_USER',
                        passwordVariable: 'DH_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DH_PASSWORD" | docker login \
                          -u "$DH_USER" --password-stdin

                        docker push ${IMAGE_REPO}:${BUILD_NUMBER}
                        docker push ${IMAGE_REPO}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy on EC2') {
            steps {
                sh '''
                    docker pull ${IMAGE_REPO}:${BUILD_NUMBER}

                    docker rm -f ${CONTAINER_NAME} || true

                    docker run -d \
                      --name ${CONTAINER_NAME} \
                      --restart unless-stopped \
                      --memory=512m \
                      --cpus=0.5 \
                      -p 80:8080 \
                      ${IMAGE_REPO}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    for i in $(seq 1 12); do
                        if curl --fail --silent http://127.0.0.1/health > /dev/null; then
                            echo "Kanban Dashboard is healthy!"
                            exit 0
                        fi
                        echo "Waiting for application to start..."
                        sleep 5
                    done

                    echo "Application health check failed"
                    docker logs ${CONTAINER_NAME}
                    exit 1
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Kanban Dashboard deployed successfully!'
        }

        failure {
            echo 'FAILURE: CI/CD pipeline failed. Check the console output.'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}
