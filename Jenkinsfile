pipeline {
    agent any

    parameters {
        string(name: 'VERSION', description: 'Release version to deploy (ex: 1.0.0)')
    }

    environment {
        NEXUS_URL   = "http://172.31.42.87:8081"
        GROUP_ID    = "com.example.maven-project"
        ARTIFACT_ID = "webapp"

        DEV_SERVER  = "172.31.9.86"
        DEPLOY_PATH = "/opt/tomcat/webapps/"
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

        // ✅ FIXED
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

                    echo "📦 Checking artifact..."

                    if [ ! -s ${ARTIFACT_ID}-${VERSION}.war ]; then
                        echo "❌ Artifact download failed or empty"
                        exit 1
                    fi

                    ls -lh
                    '''
                }
            }
        }

        // ✅ FIXED
        stage('Deploy to DEV Server') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'docker-server-ssh',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh """
                    echo "🚀 Deploying to DEV server..."

                    # Copy artifact
                    scp -i \$SSH_KEY -o StrictHostKeyChecking=no \
                    ${ARTIFACT_ID}-${VERSION}.war \
                    \$SSH_USER@${DEV_SERVER}:${DEPLOY_PATH}

                    # Remote deployment
                    ssh -i \$SSH_KEY -o StrictHostKeyChecking=no \
                    \$SSH_USER@${DEV_SERVER} << 'EOF'

                        echo "🛑 Stopping Tomcat..."
                        sudo systemctl stop tomcat || true

                        echo "🧹 Cleaning old deployment..."
                        rm -rf /opt/tomcat/webapps/webapp*

                        echo "📦 Deploying new WAR..."
                        mv /opt/tomcat/webapps/webapp-${VERSION}.war /opt/tomcat/webapps/webapp.war

                        echo "🚀 Starting Tomcat..."
                        sudo systemctl start tomcat

                        echo "✅ Deployment completed on DEV"

                    EOF
                    """
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
