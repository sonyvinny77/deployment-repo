pipeline {
    agent any

    parameters {
        string(name: 'APP_VERSION', description: 'Docker image tag to deploy')
    }

    environment {
        DOCKER_IMAGE = "sony9014/mydeploy"
        PREPROD_SERVER = "172.31.1.212"
        CONTAINER_NAME = "myyapp"
        PORT = "8080"
    }

    stages {

        stage('Deploy to PREPROD') {
            steps {
                sshagent(['docker-server-ssh']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ec2-user@${PREPROD_SERVER} "
                        docker pull ${DOCKER_IMAGE}:${APP_VERSION} &&
                        docker stop ${CONTAINER_NAME} || true &&
                        docker rm ${CONTAINER_NAME} || true &&
                        docker run -d -p ${PORT}:8080 --name ${CONTAINER_NAME} ${DOCKER_IMAGE}:${APP_VERSION}
                    "
                    """
                }
            }
        }

        stage('Trigger PROD Deployment') {
            steps {
                build job: 'deployment-repo/prod',
                wait: false,
                parameters: [
                    string(name: 'APP_VERSION', value: "${APP_VERSION}")
                ]
            }
        }
    }

    post {
        success { echo "✅ PREPROD Deployment Successful" }
        failure { echo "❌ PREPROD Deployment Failed" }
    }
}
