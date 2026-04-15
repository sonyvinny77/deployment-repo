pipeline {
    agent any

    parameters {
        string(name: 'APP_VERSION', description: 'Docker image version to deploy')
    }

    environment {
        DOCKER_IMAGE = "sony9014/mydeploy"
        CONTAINER_NAME = "app"
        DEV_SERVER = "3.128.200.157"
    }

    stages {

        stage('Validate Input') {
            steps {
                script {
                    if (!params.APP_VERSION) {
                        error "APP_VERSION is required!"
                    }
                    echo "Deploying Version: ${params.APP_VERSION}"
                }
            }
        }

        stage('Deploy to Dev Server') {
            steps {
                script {

                    sshagent(credentials: ['docker-server-ssh']) {

                        sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@${DEV_SERVER} "

                        echo 'Pulling latest image...'
                        docker pull ${DOCKER_IMAGE}:${params.APP_VERSION}

                        echo 'Stopping old container...'
                        docker stop ${CONTAINER_NAME} || true

                        echo 'Removing old container...'
                        docker rm ${CONTAINER_NAME} || true

                        echo 'Starting new container...'
                        docker run -d -p 8084:8080 --name ${CONTAINER_NAME} ${DOCKER_IMAGE}:${params.APP_VERSION}

                        echo 'Deployment completed successfully'
                        "
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Dev Deployment Successful"
        }

        failure {
            echo "❌ Dev Deployment Failed"
        }
    }
}
