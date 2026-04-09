pipeline {
    agent any

    parameters { string(name: 'VERSION', description: 'Release version to deploy') }

    environment {
        NEXUS_URL   = "http://172.31.42.87:8081"
        GROUP_ID    = "com.example.maven-project"
        ARTIFACT_ID = "webapp"

        QA_SERVER   = "172.31.9.251"
        DEPLOY_PATH = "/opt/tomcat/webapps/"
    }

    stages {
        stage('Validate Input') {
            steps {
                script {
                    if (!params.VERSION) { error "❌ VERSION is required!" }
                    if (params.VERSION.contains("SNAPSHOT")) { error "❌ SNAPSHOT not allowed!" }
                    env.VERSION = params.VERSION
                    echo "🚀 Deploying Version: ${VERSION} to QA"
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

        stage('Deploy to QA Server') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'docker-server-ssh',
                                                   keyFileVariable: 'SSH_KEY',
                                                   usernameVariable: 'SSH_USER')]) {
                    sh """
                    echo "🚀 Copying WAR to QA server..."
                    scp -i \$SSH_KEY -o StrictHostKeyChecking=no \${ARTIFACT_ID}-\$VERSION.war \$SSH_USER@$QA_SERVER:\$DEPLOY_PATH
                    echo "✅ Deployment completed on QA"
                    """
                }
            }
        }

        stage('Trigger PREPROD Deployment') {
            steps {
                build job: 'deployment-repo/preprod',
                      wait: false,
                      parameters: [string(name: 'VERSION', value: "${VERSION}")]
            }
        }
    }

    post {
        success { echo "✅ QA Deployment Successful" }
        failure { echo "❌ QA Deployment Failed" }
    }
}
