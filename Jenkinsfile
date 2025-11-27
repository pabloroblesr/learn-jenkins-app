pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        APP_NAME='learnjenkinsapp'
        AWS_DEFAULT_REGION = 'us-west-2'
        AWS_DOCKER_REGISTRY = '703420526575.dkr.ecr.us-west-2.amazonaws.com'
        AWS_ECS_CLUSTER = 'learnJenkinsapp-Cluster-Prod'
        AWS_ECS_SERVICE = 'learnJenkinsapp-service-TD-Prod'
        AWS_ECS_TASK_DEF = 'LearnJenkinsApp-TaskDefinition-Prod'

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

        stage('Build Docker image') {
            agent{
                docker{
                    image 'my-aws-cli'
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                    reuseNode true
                }
            }
            steps{
                sh '''
                    docker build  -t $APP_NAME:$REACT_APP_VERSION .
                    '''
            }
        }

       stage('Deploy to AWS'){
            agent{
                docker{
                    image 'my-aws-cli'
                    args "--entrypoint=''"
                    reuseNode true
                }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                        echo "this the latest : $LATEST_TD_REVISION" 
                        #aws ecs update-service --cluster learnJenkinsapp-Cluster-Prod --service learnJenkinsapp-service-TD-Prod  --task-definition LearnJenkinsApp-TaskDefinition-Prod:$LATEST_TD_REVISION
                        aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE --task-definition $AWS_ECS_TASK_DEF:$LATEST_TD_REVISION
                        aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE

                    '''
                }
            }
        }
    }
}