pipeline {
    agent any
    environment{
        NETLIFY_SITE_ID = "42bf050c-7b06-4f48-8ddb-fc65275d3eef"
        NETLIFY_AUTH_TOKEN = credentials('jenkins-token')
        REACT_APP_VERSION ="1.0.$BUILD_ID"
    }
    stages {
        stage('BUild') {
            agent{
                docker{
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
        stage("AWS") {
            agent {
                docker {
                    image 'amazon/aws-cli:latest'
                    args "--entrypoint=''"
                    reuseNode true
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-creds', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                sh '''
                aws s3 sync build s3://myfirstbuckettodevlopmypreejuslifewithme
                '''
                }
                
            }
        }
        stage('Run Parallel') {
            parallel{
                stage('Test') {
                    agent{
                        docker{
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
                        always{
                            junit 'jest-results/junit.xml'
                        }
                    }
                }
                
                stage('E2E') {
                    agent{
                        docker{
                        image 'my-setup'
                        reuseNode true
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
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PlayWright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }  
                }
            }
        }
        stage('Local E2E') {
            agent{
                docker{
                image 'my-setup'
                reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = "STAGE_URL"
            }
        
            steps {
                sh '''
                    

                    netlify --version
                    echo "Deployment is started in site with site id : $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > json_output.json
                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' json_output.json)
                    npx playwright test --reporter=html

                '''
            }
            post {
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PlayWright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }  
        }
        stage('Production E2E') {
            agent{
                docker{
                image 'my-setup'
                reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = 'STAGE_URL'
            }
        
            steps {
                sh '''
                    
                    netlify --version
                    echo "Deployment is started in site with site id : $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod --json > json_output.json
                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' json_output.json)
                    npx playwright test --reporter=html
                '''
            }
            post {
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PlayWright Production Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }  
        }
                
        
        
        
    } 
}