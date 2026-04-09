pipeline {
    agent any

    parameters {
        string(name: 'APP_VERSION', description: 'Docker image version to deploy')
    }

    environment {
        DOCKER_IMAGE = "sony9014/mydeploy"
        CONTAINER_NAME = "app"
        QA_SERVER = "3.144.107.40"
    }

    stages {

        stage('Validate Input') {
            steps {
                script {
                    if (!params.APP_VERSION) {
                        error "APP_VERSION is required!"
                    }
                    echo "Deploying Version to QA: ${params.APP_VERSION}"
                }
            }
        }

        stage('Deploy to QA Server') {
            steps {
                script {

                    sshagent(credentials: ['docker-server-ssh']) {

                        sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@${QA_SERVER} "

                        echo 'Pulling image...'
                        docker pull ${DOCKER_IMAGE}:${params.APP_VERSION}

                        echo 'Stopping old container...'
                        docker stop ${CONTAINER_NAME} || true

                        echo 'Removing old container...'
                        docker rm ${CONTAINER_NAME} || true

                        echo 'Starting new container...'
                        docker run -d -p 8080:8080 --name ${CONTAINER_NAME} ${DOCKER_IMAGE}:${params.APP_VERSION}

                        echo 'QA Deployment successful'
                        "
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ QA Deployment Successful"
        }

        failure {
            echo "❌ QA Deployment Failed"
        }
    }
}
