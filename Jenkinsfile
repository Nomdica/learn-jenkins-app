pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        APP_NAME = "learnjenkinsapp"
        AWS_DEFAULT_REGION = "ap-south-1"
        AWS_DOCKER_REGISTRY = "438465137004.dkr.ecr.ap-south-1.amazonaws.com"
        AWS_ECS_CLUSTER = "LearnJenkinsApp-Cluster-Prod"
        AWS_ECS_SERVICE = "LearnJenkinsApp-Service-Prod"
        AWS_ECS_TASK_DEFINITION = "LearnJenkinsApp-TaskDefinition-Prod"
    }

    stages {

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

        stage('Buil Docker Image') {
            agent{
                docker {
                    image 'my-aws-cli'
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint ''"
                    reuseNode true
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                sh '''
                    docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION .
                    aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
                    docker push $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION
                '''
                }
            }
        }

        stage('Deploy to AWS'){
            agent{
                docker {
                    image 'my-aws-cli'
                    args "--entrypoint ''"
                    reuseNode true
                }
            }
            // environment {
            // }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                sh '''
                    aws --version
                    sed -i "s/#APP_VERSION#/$REACT_APP_VERSION/g" aws/task-definition-prod.json
                    LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq -r '.taskDefinition.revision')
                    echo $LATEST_TD_REVISION
                    aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE --task-definition $AWS_ECS_TASK_DEFINITION:$LATEST_TD_REVISION
                    aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE
                '''
                }
            }
        }
    }
}
