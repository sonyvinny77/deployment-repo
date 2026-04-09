pipeline {
    agent any

    parameters {
        string(name: 'APP_VERSION', description: 'Docker image tag to deploy')
    }

    environment {
        DOCKER_IMAGE = "sony9014/mydeploy"
        QA_SERVER    = "172.31.9.251"
        CONTAINER_NAME = "myyapp"
        PORT = "8080"
    }

    stages {

        stage('Deploy to QA') {
            steps {
                sshagent(['docker-server-ssh']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ec2-user@${QA_SERVER} "
                        docker pull ${DOCKER_IMAGE}:${APP_VERSION} &&
                        docker stop ${CONTAINER_NAME} || true &&
                        docker rm ${CONTAINER_NAME} || true &&
                        docker run -d -p ${PORT}:8080 --name ${CONTAINER_NAME} ${DOCKER_IMAGE}:${APP_VERSION}
                    "
                    """
                }
            }
        }

        stage('Trigger PREPROD Deployment') {
            steps {
                build job: 'deployment-repo/preprod',
                wait: false,
                parameters: [
                    string(name: 'APP_VERSION', value: "${APP_VERSION}")
                ]
            }
        }
    }

    post {
        success { echo "✅ QA Deployment Successful" }
        failure { echo "❌ QA Deployment Failed" }
    }
}
