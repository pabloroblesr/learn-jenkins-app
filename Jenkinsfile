pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        AWS_DEFAULT_REGION = 'us-west-2'
        AWS_ECS_CLUSTER = 'learnJenkinsapp-Cluster-Prod'
        AWS_ECS_SERVICE = 'learnJenkinsapp-service-TD-Prod'
        AWS_ECS_TASK_DEF = 'learnJenkinsapp-TaskDefinition-Prod'

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
            steps{
                sh 'docker build -t myjenkinsapp .'
            }
        }
        stage('Deploy to AWS'){
            agent{
                docker{
                    image 'amazon/aws-cli'
                    args "-u root --entrypoint=''"
                    reuseNode true
                }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        #yum install jq -y
                        #aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                        echo "this the latest : $LATEST_TD_REVISION" 
                        aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE --task-definition $AWS_ECS_TASK_DEF:$LATEST_TD_REVISION
                        aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE

                    '''
                }
            }
        }
    }
}