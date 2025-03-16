pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        AWS_DEFAULT_REGION = "ap-south-1"
    }

    stages {

        stage('Deploy to AWS'){
            agent{
                docker {
                    image 'amazon/aws-cli'
                    args "-u root --entrypoint ''"
                    reuseNode true
                }
            }
            // environment {
            // }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                sh '''
                    aws --version
                    yum install -y jq
                    LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq -r '.taskDefinition.revision')
                    echo $LATEST_TD_REVISION
                    aws ecs update-service --cluster LearnJenkinsApp-Cluster-Prod --service LearnJenkinsApp-Service-Prod --task-definition LearnJenkinsApp-Task-Definition-Prod:$LATEST_TD_REVISION
                '''
                }
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
    }
}
