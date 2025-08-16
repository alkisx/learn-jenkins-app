pipeline {
    agent any

    environment {
       NETLIFY_SITE_ID = 'a22cc808-5d50-4b4c-946d-8938b7a2e6eb'
       NETLIFY_AUTH_TOKEN = credentials('netlify-token')
       REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {       
        // required for first time build
        // stage('Docker') {
        //     steps {
        //         sh 'docker build -t my-playwright .'
        //     }
        // }


        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli:2.28.11'
                    args  "--entrypoint=''" // --entrypoint=''
                }
            }

            steps {
                sh '''
                    aws --version
                    aws s3 ls
                '''
            }
        }

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
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    steps {
                        echo "Test stage"
                        sh '''
                            serve -s build &
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
                echo "Test stage"
                sh '''
                    netlify --version
                    
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"                 
                    netlify status
                                        
                    netlify deploy --dir=build --no-build --json > deploy-output.json
                                             
                    CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deploy-output.json)                     
                    
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

        stage('Deply Prod') {
            agent {
                // this image already includes NodeJS:
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                 CI_ENVIRONMENT_URL = 'https://ornate-creponne-3d63e0.netlify.app'
            }

            steps {
                echo "Test stage"
                sh '''
                    node --version                
                    netlify --version
                    echo "Deploying to production. Project ID: $NETLIFY_SITE_ID" 
                    netlify status
                    netlify deploy --dir=build --prod --no-build
                    # Adding some sleep time to ensure the deployment is complete
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
