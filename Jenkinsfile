pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'hiran4024'
        IMAGE_REPO = 'hiran4024/kanban-dashboard'
        CONTAINER_NAME = 'kanban-dashboard'
        DEPLOYMENT_STARTED = 'false'
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
                script {
                    sh '''
                        # Save the currently deployed image
                        if docker inspect ${CONTAINER_NAME} >/dev/null 2>&1; then
                            docker inspect \
                              --format '{{.Image}}' \
                              ${CONTAINER_NAME} > previous-image.txt

                            echo "Previous image saved:"
                            cat previous-image.txt
                        else
                            : > previous-image.txt
                            echo "No previous deployment found."
                        fi

                        # Pull the new image
                        docker pull ${IMAGE_REPO}:${BUILD_NUMBER}

                        # Remove any leftover candidate
                        docker rm -f ${CONTAINER_NAME}-candidate || true

                        # Start candidate on temporary port 8081
                        docker run -d \
                          --name ${CONTAINER_NAME}-candidate \
                          --restart unless-stopped \
                          --memory=512m \
                          --cpus=0.5 \
                          -p 8081:8080 \
                          ${IMAGE_REPO}:${BUILD_NUMBER}

                        # Validate candidate before switching
                        for i in $(seq 1 12); do
                            if curl --fail --silent \
                              http://127.0.0.1:8081/health > /dev/null; then
                                echo "Candidate is healthy!"
                                exit 0
                            fi

                            echo "Waiting for candidate..."
                            sleep 5
                        done

                        echo "Candidate health check failed."
                        docker logs ${CONTAINER_NAME}-candidate
                        exit 1
                    '''

                    # Mark production replacement as started
                    env.DEPLOYMENT_STARTED = 'true'

                    sh '''
                        # Replace the production container
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
        }

        stage('Health Check') {
            steps {
                sh '''
                    for i in $(seq 1 12); do
                        if curl --fail --silent \
                          http://127.0.0.1/health > /dev/null; then
                            echo "Kanban Dashboard is healthy!"
                            exit 0
                        fi

                        echo "Waiting for application..."
                        sleep 5
                    done

                    echo "Production health check failed."
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
            echo 'FAILURE: Deployment failed. Checking rollback...'

            script {
                if (env.DEPLOYMENT_STARTED == 'true') {
                    sh '''
                        echo "Starting rollback..."

                        docker rm -f ${CONTAINER_NAME} || true

                        if [ -s previous-image.txt ]; then
                            PREVIOUS_IMAGE=$(cat previous-image.txt)

                            echo "Restoring previous image: $PREVIOUS_IMAGE"

                            docker run -d \
                              --name ${CONTAINER_NAME} \
                              --restart unless-stopped \
                              --memory=512m \
                              --cpus=0.5 \
                              -p 80:8080 \
                              "$PREVIOUS_IMAGE"

                            for i in $(seq 1 12); do
                                if curl --fail --silent \
                                  http://127.0.0.1/health > /dev/null; then
                                    echo "ROLLBACK SUCCESSFUL!"
                                    exit 0
                                fi

                                echo "Waiting for restored application..."
                                sleep 5
                            done

                            echo "ROLLBACK FAILED: Previous image did not become healthy."
                            exit 1
                        else
                            echo "No previous image available for rollback."
                            exit 1
                        fi
                    '''
                } else {
                    echo 'Production was not replaced. Existing application remains untouched.'
                }
            }
        }

        always {
            sh '''
                docker rm -f ${CONTAINER_NAME}-candidate || true
            '''

            echo 'Pipeline execution completed.'
        }
    }
}
