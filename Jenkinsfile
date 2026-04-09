pipeline {
    agent any

    parameters {
        string(name: 'APP_VERSION', description: 'Docker image tag to deploy (ex: 1.0.0)')
    }

    environment {
        DOCKER_IMAGE = "sony9014/mydeploy"
        DEV_SERVER   = "172.31.9.86"
        CONTAINER_NAME = "myyapp"
        PORT = "8080"
        NEXUS_URL   = "http://172.31.42.87:8081"
        GROUP_ID    = "com.example.maven-project"
        ARTIFACT_ID = "webapp"
        DEPLOY_PATH = "/opt/tomcat/webapps/
    }

    stages {

        stage('Validate Input') {
            steps {
                script {
                    if (!params.APP_VERSION) {
                        error "❌ APP_VERSION is required!"
                    }
                    env.APP_VERSION = params.APP_VERSION
                    echo "🚀 Deploying ${DOCKER_IMAGE}:${APP_VERSION} to DEV"
                }
            }
        }
        stage('Download Artifact from Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-creds',
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {
                    sh '''
                        echo "⬇️ Downloading artifact from Nexus..."
                        GROUP_PATH=$(echo $GROUP_ID | tr '.' '/')
                        curl -f -u $NEXUS_USER:$NEXUS_PASS -O \
                            $NEXUS_URL/repository/maven-releases/$GROUP_PATH/$ARTIFACT_ID/$VERSION/${ARTIFACT_ID}-${VERSION}.war
                        ls -lh
                    '''
                }
            }
        }

        stage('Deploy to DEV') {
            steps {
                sshagent(['docker-server-ssh']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ec2-user@${DEV_SERVER} "
                        docker pull ${DOCKER_IMAGE}:${APP_VERSION} &&
                        docker stop ${CONTAINER_NAME} || true &&
                        docker rm ${CONTAINER_NAME} || true &&
                        docker run -d -p ${PORT}:8080 --name ${CONTAINER_NAME} ${DOCKER_IMAGE}:${APP_VERSION}
                    "
                    """
                }
            }
        }

        stage('Trigger QA Deployment') {
            steps {
                build job: 'deployment-repo/qa',
                wait: false,
                parameters: [
                    string(name: 'APP_VERSION', value: "${APP_VERSION}")
                ]
            }
        }
    }

    post {
        success { echo "✅ DEV Deployment Successful" }
        failure { echo "❌ DEV Deployment Failed" }
    }
}
