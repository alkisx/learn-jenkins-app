pipeline {
    agent any

    environment {
       NETLIFY_SITE_ID = 'a22cc808-5d50-4b4c-946d-8938b7a2e6eb'
       NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:22-alpine3.21'
                    reuseNode true
                }
            }
            
            steps {
                sh '''
                    echo  'Small change'
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
            
        }

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:22-alpine3.21'
                            reuseNode true
                        }
                    }
                    steps {
                        echo "Test stage"
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
                            image 'mcr.microsoft.com/playwright:v1.53.0-noble'
                            reuseNode true
                        }
                    }
                    steps {
                        echo "Test stage"
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            # wait 10 seconds for the server to start:
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([
                                allowMissing: false, 
                                alwaysLinkToLastBuild: false, 
                                icon: '', 
                                keepAll: false, 
                                reportDir: 'playwright-report', 
                                reportFiles: 'index.html', 
                                reportName: 'Playwright Local Report', 
                                reportTitles: '', 
                                useWrapperFileDirectly: true
                            ])
                        }
                    }
                }
            }
        }

        stage('Deploy staging') {
            agent {
                docker {
                    image 'node:22-alpine3.21'
                    reuseNode true
                }
            }
            
            steps {
                sh '''
                   npm install netlify-cli node-jq
                   node_modules/.bin/netlify --version
                   echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID" 
                   node_modules/.bin/netlify status
                   node_modules/.bin/netlify deploy --dir=build --no-build --json > deploy-output.json 
                   #node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json
                '''

                script {
                    env.STAGING_URL = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
                }
            }
            
        }

        stage('Staging E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.53.0-noble'
                    reuseNode true
                }
            }

            environment {
                 CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
            }

            steps {
                echo "Test stage"
                sh '''
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([
                        allowMissing: false, 
                        alwaysLinkToLastBuild: false, 
                        icon: '', 
                        keepAll: false, 
                        reportDir: 'playwright-report', 
                        reportFiles: 'index.html', 
                        reportName: 'Staging E2E Report', 
                        reportTitles: '', 
                        useWrapperFileDirectly: true
                    ])
                }
            }
        }

        stage('Approval') {
            steps{
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                }
                
            }
            
        }

        stage('Deploy prod') {
            agent {
                docker {
                    image 'node:22-alpine3.21'
                    reuseNode true
                }
            }
            
            steps {
                sh '''
                   npm install netlify-cli
                   node_modules/.bin/netlify --version
                   echo "Deploying to production. Project ID: $NETLIFY_SITE_ID" 
                   node_modules/.bin/netlify status
                   node_modules/.bin/netlify deploy --dir=build --prod --no-build
                '''
            }
            
        }

        stage('Prod E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.53.0-noble'
                    reuseNode true
                }
            }

            environment {
                 CI_ENVIRONMENT_URL = 'https://ornate-creponne-3d63e0.netlify.app'
            }

            steps {
                echo "Test stage"
                sh '''
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([
                        allowMissing: false, 
                        alwaysLinkToLastBuild: false, 
                        icon: '', 
                        keepAll: false, 
                        reportDir: 'playwright-report', 
                        reportFiles: 'index.html', 
                        reportName: 'Prod E2E Report', 
                        reportTitles: '', 
                        useWrapperFileDirectly: true
                    ])
                }
            }
        }

        


    }


    

}
