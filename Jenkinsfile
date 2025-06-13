pipeline {
    agent any
    environment{
        NETLIFY_SITE_ID = "42bf050c-7b06-4f48-8ddb-fc65275d3eef"
        NETLIFY_AUTH_TOKEN = credentials('jenkins-token')
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
                            npx playwright test  --reporter=html
                        '''
                    }
                    post {
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PlayWright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }  
                }
                
            }
        }
        
    } 
}