pipeline {
    agent any

    parameters {
        string(name: 'VERSION', description: 'Release version to deploy (ex: 1.0.0)')
    }

    environment {
        NEXUS_URL   = "http://172.31.42.87:8081"
        GROUP_ID    = "com.example.maven-project"
        ARTIFACT_ID = "webapp"

        // DEV ENV DETAILS
        DEV_SERVER  = "172.31.9.86"          // change this
        DEPLOY_PATH = "/opt/tomcat/webapps/"  // change if needed
    }

    stages {

        stage('Validate Input') {
            steps {
                script {
                    if (!params.VERSION) {
                        error "❌ VERSION parameter is required!"
                    }

                    if (params.VERSION.contains("SNAPSHOT")) {
                        error "❌ SNAPSHOT not allowed in deployment!"
                    }

                    env.VERSION = params.VERSION
                    echo "✅ Deploying Version: ${VERSION}"
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

                    curl -u $NEXUS_USER:$NEXUS_PASS -O \
                    $NEXUS_URL/repository/maven-releases/$GROUP_PATH/$ARTIFACT_ID/$VERSION/${ARTIFACT_ID}-${VERSION}.war

                    ls -l
                    '''
                }
            }
        }

        stage('Deploy to DEV Server') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'docker-server-ssh',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh '''
                    echo "🚀 Deploying to DEV server..."

                    scp -i $SSH_KEY -o StrictHostKeyChecking=no \
                    ${ARTIFACT_ID}-${VERSION}.war \
                    $SSH_USER@$DEV_SERVER:$DEPLOY_PATH

                    ssh -i $SSH_KEY -o StrictHostKeyChecking=no \
                    $SSH_USER@$DEV_SERVER << EOF

                    echo "Restarting Tomcat..."
                    sudo systemctl restart tomcat

                    echo "Deployment completed on DEV"
                    EOF
                    '''
                }
            }
        }
        stage('Trigger QA Deployment') {
            steps {
                build job: 'deployment-repo/qa',
                wait: false,
                parameters: [
                    string(name: 'VERSION', value: "${VERSION}")
                ]
            }
        }
    }

    post {
        success {
            echo "✅ DEV Deployment Successful"
        }
        failure {
            echo "❌ DEV Deployment Failed"
        }
    }
}
