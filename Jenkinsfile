pipeline {
    agent any

    parameters {
        string(name: 'VERSION', description: 'Artifact version')
    }

    environment {
        NEXUS_URL = "http://172.31.42.87:8081"
        GROUP_ID = "com.example.maven-project"
        ARTIFACT_ID = "webapp"
        ENV = "dev"
    }

    stages {

        stage('Download Artifact') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh '''
                    GROUP_PATH=$(echo $GROUP_ID | tr '.' '/')

                    curl -u $USER:$PASS -O \
                    $NEXUS_URL/repository/maven-releases/$GROUP_PATH/$ARTIFACT_ID/$VERSION/${ARTIFACT_ID}-${VERSION}.war
                    '''
                }
            }
        }

        stage('Deploy to DEV') {
            steps {
                sh '''
                echo "Deploying to DEV environment"
                # Example:
                # scp war to server
                # restart service
                '''
            }
        }

        stage('Smoke Test') {
            steps {
                sh "echo Running smoke test on DEV"
            }
        }

        stage('Trigger QA') {
            steps {
                build job: 'deployment-repo/qa', wait: false, parameters: [
                    string(name: 'VERSION', value: "${VERSION}")
                ]
            }
        }
    }
}
