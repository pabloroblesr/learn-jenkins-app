pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        AWS_DEFAULT_REGION = 'us-west-2'

    }

    stages {
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
                        aws ecs update-service --cluster learnJenkinsapp-Cluster-Prod --service learnJenkinsapp-service-TD-Prod --task-definition learnJenkinsapp-TaskDefinition-Prod:$LATEST_TD_REVISION
                        aws ecs wait services-stable --cluster learnJenkinsapp-Cluster-Prod --services learnJenkinsapp-service-TD-Prod
                        
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