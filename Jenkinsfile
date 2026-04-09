pipeline {
    agent any

    parameters {
        string(name: 'VERSION', description: 'Release version to deploy')
    }

    environment {
        NEXUS_URL   = "http://172.31.42.87:8081"
        GROUP_ID    = "com.example.maven-project"
        ARTIFACT_ID = "webapp"

        PROD_SERVER = "172.31.5.86"
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

                    echo "📦 Validating artifact..."

                    if [ ! -s ${ARTIFACT_ID}-${VERSION}.war ]; then
                        echo "❌ Artifact download failed or empty"
                        exit 1
                    fi

                    ls -lh
                    '''
                }
            }
        }

        // 🔥 PRODUCTION DEPLOYMENT WITH BACKUP + ROLLBACK
        stage('Deploy to PROD Server') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'docker-server-ssh',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh """
                    echo "🚀 Deploying to PROD server..."

                    scp -i \$SSH_KEY -o StrictHostKeyChecking=no \
                    ${ARTIFACT_ID}-${VERSION}.war \
                    \$SSH_USER@${PROD_SERVER}:${DEPLOY_PATH}

                    ssh -i \$SSH_KEY -o StrictHostKeyChecking=no \
                    \$SSH_USER@${PROD_SERVER} << 'EOF'

                        echo "🛑 Stopping Tomcat..."
                        sudo systemctl stop tomcat || true

                        echo "💾 Taking backup..."
                        if [ -f /opt/tomcat/webapps/webapp.war ]; then
                            cp /opt/tomcat/webapps/webapp.war /opt/tomcat/webapps/webapp_backup.war
                        fi

                        echo "🧹 Cleaning old deployment..."
                        rm -rf /opt/tomcat/webapps/webapp*

                        echo "📦 Deploying new WAR..."
                        mv /opt/tomcat/webapps/webapp-${VERSION}.war /opt/tomcat/webapps/webapp.war

                        echo "🚀 Starting Tomcat..."
                        sudo systemctl start tomcat

                        echo "✅ Deployment step completed"

                    EOF
                    """
                }
            }
        }
    }

    // 🔥 AUTO ROLLBACK
    post {
        success {
            echo "🎉 PROD Deployment Successful"
        }

        failure {
            echo "❌ Deployment failed! Starting rollback..."

            withCredentials([sshUserPrivateKey(
                credentialsId: 'docker-server-ssh',
                keyFileVariable: 'SSH_KEY',
                usernameVariable: 'SSH_USER'
            )]) {
                sh """
                ssh -i \$SSH_KEY -o StrictHostKeyChecking=no \
                \$SSH_USER@${PROD_SERVER} << 'EOF'

                    echo "🔁 Rolling back..."

                    sudo systemctl stop tomcat || true

                    if [ -f /opt/tomcat/webapps/webapp_backup.war ]; then
                        rm -rf /opt/tomcat/webapps/webapp*
                        mv /opt/tomcat/webapps/webapp_backup.war /opt/tomcat/webapps/webapp.war
                    fi

                    sudo systemctl start tomcat

                    echo "✅ Rollback completed"

                EOF
                """
            }
        }
    }
}
