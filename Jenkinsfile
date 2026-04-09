pipeline {
    agent any

    parameters {
        string(name: 'VERSION', description: 'Release version to deploy')
    }

    environment {
        NEXUS_URL   = "http://172.31.42.87:8081"
        GROUP_ID    = "com.example.maven-project"
        ARTIFACT_ID = "webapp"

        PROD_SERVER = "172.31.5.86"        // change this
        DEPLOY_PATH = "/opt/tomcat/webapps/"
    }

    stages {

        stage('Validate Input') {
            steps {
                script {
                    if (!params.VERSION) {
                        error "❌ VERSION is required!"
                    }

                    if (params.VERSION.contains("SNAPSHOT")) {
                        error "❌ SNAPSHOT not allowed!"
                    }

                    env.VERSION = params.VERSION
                    echo "🚀 Deploying Version: ${VERSION} to PROD"
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
                    GROUP_PATH=$(echo $GROUP_ID | tr '.' '/')

                    curl -u $NEXUS_USER:$NEXUS_PASS -O \
                    $NEXUS_URL/repository/maven-releases/$GROUP_PATH/$ARTIFACT_ID/$VERSION/${ARTIFACT_ID}-${VERSION}.war
                    '''
                }
            }
        }

        stage('Deploy to PROD Server') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'docker-server-ssh',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh '''
                    scp -i $SSH_KEY -o StrictHostKeyChecking=no \
                    ${ARTIFACT_ID}-${VERSION}.war \
                    $SSH_USER@$PROD_SERVER:$DEPLOY_PATH

                    ssh -i $SSH_KEY -o StrictHostKeyChecking=no \
                    $SSH_USER@$PROD_SERVER << EOF
                    sudo systemctl restart tomcat
                    EOF
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "🎉 PROD Deployment Successful"
        }
        failure {
            echo "❌ PROD Deployment Failed"
        }
    }
}
