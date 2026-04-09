pipeline {
    agent any

    parameters {
        string(name: 'APP_VERSION', description: 'Docker image tag to deploy')
    }

    environment {
        DOCKER_IMAGE = "sony9014/mydeploy"
        PROD_SERVER  = "172.31.5.86"
        CONTAINER_NAME = "myyapp"
        PORT = "8080"
    }

    stages {

        stage('Deploy to PROD') {
            steps {
                sshagent(['docker-server-ssh']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ec2-user@${PROD_SERVER} "
                        docker pull ${DOCKER_IMAGE}:${APP_VERSION} &&
                        docker stop ${CONTAINER_NAME} || true &&
                        docker rm ${CONTAINER_NAME} || true &&
                        docker run -d -p ${PORT}:8080 --name ${CONTAINER_NAME} ${DOCKER_IMAGE}:${APP_VERSION}
                    "
                    """
                }
            }
        }
    }

    post {
        success { echo "✅ PROD Deployment Successful" }
        failure { echo "❌ PROD Deployment Failed" }
    }
}
