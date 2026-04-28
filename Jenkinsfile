pipeline {
    agent any

    options {
        timeout(time: 2, unit: 'MINUTES')
    }

    environment {
        NEXUS_URL = "http://localhost:8083"
        CREDENTIALS_ID = "nexus-docker-publisher"
        IMAGE_NAME = "sumador"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        NEXUS_HOST = "localhost:8083"
        NEXUS_REPO = "repository/myrepo"
        ARTIFACT_ID = "elbuo8/webapp:${env.BUILD_NUMBER}"
        NEXUS_IMAGE = "${NEXUS_HOST}/${NEXUS_REPO}/${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh """
                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Run tests') {
            steps {
                sh "docker run --rm ${IMAGE_NAME}:${IMAGE_TAG} npm test"
            }
        }

        stage('Tag Docker Image') {
            steps {
                echo "Tagging Docker image for Nexus repository..."
                sh """
                docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${NEXUS_IMAGE}
                """
            }
        }

        stage('Deploy Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: CREDENTIALS_ID, usernameVariable: 'NEXUS_USERNAME', passwordVariable: 'NEXUS_PASSWORD')]) {
                    sh '''
                    set +x
                    TARGET_HOST="$NEXUS_HOST"
                    TARGET_IMAGE="$NEXUS_IMAGE"

                    if ! printf '%s' "$NEXUS_PASSWORD" | docker login "$TARGET_HOST" -u "$NEXUS_USERNAME" --password-stdin; then
                      TARGET_HOST="host.docker.internal:8083"
                      TARGET_IMAGE="$TARGET_HOST/$NEXUS_REPO/$IMAGE_NAME:$IMAGE_TAG"
                      docker tag "$IMAGE_NAME:$IMAGE_TAG" "$TARGET_IMAGE"
                      printf '%s' "$NEXUS_PASSWORD" | docker login "$TARGET_HOST" -u "$NEXUS_USERNAME" --password-stdin
                    fi

                    docker push "$TARGET_IMAGE"
                    docker logout "$TARGET_HOST" || true
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning up local Docker images..."
            sh """
            docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true
            docker rmi ${NEXUS_IMAGE} || true
            """
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check the logs for details."
        }
    }
}
