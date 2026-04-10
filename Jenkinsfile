pipeline {
    agent any

    parameters {
        string(name: 'APP_VERSION', defaultValue: '', description: 'Docker image version')
    }

    environment {
        DOCKER_IMAGE   = "sony9014/mydeploy"
        CONTAINER_NAME = "app-prod"
        PROD_SERVER    = "3.146.107.154"
    }

    stages {

        stage('Set Version') {
            steps {
                script {
                    if (!params.APP_VERSION?.trim()) {
                        error "APP_VERSION not received!"
                    }

                    env.APP_VERSION = params.APP_VERSION
                    echo "Deploying Version: ${env.APP_VERSION}"
                }
            }
        }

        stage('Deploy to PROD') {
            steps {
                script {
                    sshagent(credentials: ['docker-server-ssh']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@${PROD_SERVER} '
                        set -e

                        echo "Pulling image..."
                        docker pull ${DOCKER_IMAGE}:${APP_VERSION}

                        echo "Stopping old container..."
                        docker stop ${CONTAINER_NAME} || true

                        echo "Removing old container..."
                        docker rm ${CONTAINER_NAME} || true

                        echo "Starting new container..."
                        docker run -d -p 8083:8080 --name ${CONTAINER_NAME} ${DOCKER_IMAGE}:${APP_VERSION}

                        echo "Verifying container..."
                        docker ps | grep ${CONTAINER_NAME} || { echo "Container failed!"; exit 1; }
                        '
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ PROD Deployment Successful"
        }
        failure {
            echo "❌ PROD Deployment Failed"
        }
    }
}
