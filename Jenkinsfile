pipeline {
    agent any

    parameters { string(name: 'VERSION', description: 'Release version to deploy') }

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
                    if (!params.VERSION) { error "❌ VERSION is required!" }
                    if (params.VERSION.contains("SNAPSHOT")) { error "❌ SNAPSHOT not allowed!" }
                    env.VERSION = params.VERSION
                    echo "🚀 Deploying Version: ${VERSION} to DEV"
                }
            }
        }

        stage('Download Artifact from Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-creds',
                                                 usernameVariable: 'NEXUS_USER',
                                                 passwordVariable: 'NEXUS_PASS')]) {
                    sh """
                    echo "⬇️ Downloading artifact..."
                    GROUP_PATH=\$(echo $GROUP_ID | tr '.' '/')
                    curl -f -u \$NEXUS_USER:\$NEXUS_PASS -O \
                    \$NEXUS_URL/repository/maven-releases/\$GROUP_PATH/\$ARTIFACT_ID/\$VERSION/\${ARTIFACT_ID}-\$VERSION.war
                    ls -lh
                    """
                }
            }
        }

        stage('Deploy to DEV Server') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'docker-server-ssh',
                                                   keyFileVariable: 'SSH_KEY',
                                                   usernameVariable: 'SSH_USER')]) {
                    sh """
                    echo "🚀 Copying WAR to DEV server..."
                    scp -i \$SSH_KEY -o StrictHostKeyChecking=no \${ARTIFACT_ID}-\$VERSION.war \$SSH_USER@$DEV_SERVER:\$DEPLOY_PATH
                    echo "✅ Deployment completed on DEV"
                    """
                }
            }
        }

        stage('Trigger QA Deployment') {
            steps {
                build job: 'deployment-repo/qa',
                      wait: false,
                      parameters: [string(name: 'VERSION', value: "${VERSION}")]
            }
        }
    }

    post {
        success { echo "✅ DEV Deployment Successful" }
        failure { echo "❌ DEV Deployment Failed" }
    }
}
