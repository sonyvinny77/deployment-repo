pipeline {
    agent any

    parameters {
        string(name: 'APP_VERSION', defaultValue: '', description: 'Docker image version to deploy')
    }

    environment {
        DOCKER_IMAGE = "sony9014/mydeploy"
        CONTAINER_NAME = "app-qa"
        QA_SERVER = "3.144.107.40"
    }

    stages {

        stage('Determine Version') {
            steps {
                script {
                    if (!params.APP_VERSION?.trim()) {
                        withCredentials([usernamePassword(
                            credentialsId: 'dockerhub-creds',
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )]) {
                            env.APP_VERSION = sh(
                                script: """
                                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                                    curl -s -u \$DOCKER_USER:\$DOCKER_PASS https://hub.docker.com/v2/repositories/${DOCKER_IMAGE}/tags?page_size=100 | \
                                    jq -r '.results[].name' | sort -V | tail -n1
                                """,
                                returnStdout: true
                            ).trim()
                        }
                        echo "Auto-detected latest Docker version: ${env.APP_VERSION}"
                    } else {
                        env.APP_VERSION = params.APP_VERSION
                        echo "Using APP_VERSION from upstream: ${env.APP_VERSION}"
                    }

                    if (!env.APP_VERSION) {
                        error "No Docker version found!"
                    }
                }
            }
        }

        stage('Deploy to QA Server') {
            steps {
                script {
                    sshagent(credentials: ['docker-server-ssh']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@${QA_SERVER} '
                        set -e

                        echo "Pulling Docker image ${DOCKER_IMAGE}:${APP_VERSION} ..."
                        docker pull ${DOCKER_IMAGE}:${APP_VERSION}

                        echo "Stopping old container..."
                        docker stop ${CONTAINER_NAME} || true

                        echo "Removing old container..."
                        docker rm ${CONTAINER_NAME} || true

                        echo "Starting new container..."
                        docker run -d -p 8080:8080 --name ${CONTAINER_NAME} ${DOCKER_IMAGE}:${APP_VERSION}

                        echo "Checking if container started..."
                        docker ps | grep ${CONTAINER_NAME} || { echo "Container failed to start!"; exit 1; }

                        echo "QA Deployment completed successfully"
                        '
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
