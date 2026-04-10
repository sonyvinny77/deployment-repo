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

        stage('Deploy to PROD') {
            steps {
                script {
                    sshagent(credentials: ['docker-server-ssh']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@${PROD_SERVER} '
                        set -e

                        docker pull ${DOCKER_IMAGE}:${APP_VERSION}

                        docker stop ${CONTAINER_NAME} || true
                        docker rm ${CONTAINER_NAME} || true

                        docker run -d -p 8083:8080 --name ${CONTAINER_NAME} ${DOCKER_IMAGE}:${APP_VERSION}

                        docker ps | grep ${CONTAINER_NAME}
                        '
                        """
                    }
                }
            }
        }
    }
}
