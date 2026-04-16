pipeline {
    agent any

    parameters {
        string(name: 'APP_VERSION', defaultValue: '', description: 'Docker image version to deploy')
    }

    environment {
        DOCKER_IMAGE   = "sony9014/mydeploy"
        CONTAINER_NAME = "app-preprod"
        PREPROD_SERVER = "3.149.3.196"
    }

    stages {

        stage('Determine Version') {
            steps {
                script {
                    if (!params.APP_VERSION?.trim()) {
                        error "APP_VERSION not received from upstream!"
                    }

                    env.APP_VERSION = params.APP_VERSION
                    echo "Using APP_VERSION from upstream: ${env.APP_VERSION}"
                }
            }
        }

        stage('Deploy to PreProd Server') {
            steps {
                script {
                    sshagent(credentials: ['docker-server-ssh']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@${PREPROD_SERVER} '
                        set -e

                        echo "Pulling Docker image ${DOCKER_IMAGE}:${APP_VERSION} ..."
                        docker pull ${DOCKER_IMAGE}:${APP_VERSION}

                        echo "Stopping old container..."
                        docker stop ${CONTAINER_NAME} || true

                        echo "Removing old container..."
                        docker rm ${CONTAINER_NAME} || true

                        echo "Starting new container..."
                        docker run -d -p 8082:8080 --name ${CONTAINER_NAME} ${DOCKER_IMAGE}:${APP_VERSION}

                        echo "Checking if container started..."
                        docker ps | grep ${CONTAINER_NAME} || { echo "Container failed to start!"; exit 1; }

                        echo "PreProd Deployment completed successfully"
                        '
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ PreProd Deployment Successful"
        }
        failure {
            echo "❌ PreProd Deployment Failed"
        }
    }
}
