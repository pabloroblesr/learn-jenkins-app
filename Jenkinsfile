pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '3e7929e5-9590-49a3-9457-f969d52abeb4'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {
        stage('AWS'){
            agent{
                docker{
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        echo "hello S3!" > index.html
                        aws s3 cp index.html s3://learn-jenkins-20251115/index.html
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
       /* stage('Run Tests') {
            parallel {
                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }
                stage('E2E') {
                    agent {
                        docker {
                            image 'my-playwright'
                            reuseNode true
                            //args '-u root:root'
                        }
                    }
                    steps {
                        sh '''
                            serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        stage('Deploy Staging') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "STAGING_URL_TO_BE_SET"
            }
            steps {
                sh '''
                    netlify --version
                    echo "Deploy to Staging SITE_ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --no-build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deploy-output.json)
                    npx playwright test --reporter=html
                   
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
        stage('Deploy Prod') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://verdant-malabi-303fbf.netlify.app'
            }
            steps {
                sh '''
                    node --version
                    netlify --version
                    echo "Deploy to production SITE_ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod --no-build
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }*/
    }
}