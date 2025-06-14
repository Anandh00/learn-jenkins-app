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
                        image 'mcr.microsoft.com/playwright:v1.53.0-noble'
                        reuseNode true
                        }
                    }
                
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
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
                image 'mcr.microsoft.com/playwright:v1.53.0-noble'
                reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = "STAGE_URL"
            }
        
            steps {
                sh '''
                    
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    echo "Deployment is started in site with site id : $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > json_output.json
                    CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' json_output.json)
                    npx playwright test --reporter=html

                '''
            }
            post {
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PlayWright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }  
        }
        stage('Authentication') {
            steps {
                    echo 'Waiting for the authentication from cheif'
                    timeout(1) {
                        input cancel: 'Developers need to recheck, Abort', message: 'Ready to deploy', ok: 'I\'m approving this deployment'
                    }
            }
        }
        stage('Production E2E') {
            agent{
                docker{
                image 'mcr.microsoft.com/playwright:v1.53.0-noble'
                reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = 'STAGE_URL'
            }
        
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deployment is started in site with site id : $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod --json > json_output.json
                    CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' json_output.json)
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