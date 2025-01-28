pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = 'e3092215-119a-41d8-b9a5-853fd9c753d7'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }
    stages {
        stage('Build') {
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
        stage('Test'){
            parallel{
                stage('Unit Tests'){
                     agent{
                        docker{
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps{
                        sh'''
                            test -f build/index.html
                            npm test
                        '''
                    }
                }
                stage('E2E'){
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post{
                        always{
                            pkill -f serve  // Ensure the background server is stopped
                            publishHTML([
                                        reportDir: 'playwright-report',      // Directory where Playwright's HTML report is generated
                                        reportFiles: 'index.html',          // Main file of the HTML report
                                        reportName: 'Playwright Test Report', // Name of the report in Jenkins
                                        allowMissing: false,                // Fail the build if the report is missing
                                        alwaysLinkToLastBuild: true,        // Always link the latest report
                                        keepAll: true                       // Retain reports for all builds
                                ])
                        }
                    }
                }
            }
        }
        stage('Deploy') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to production site ID $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
    }
    post{
        always{
            junit 'test-results/junit.xml'
        }
    }
}
