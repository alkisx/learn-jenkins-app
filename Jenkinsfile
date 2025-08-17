pipeline {
    agent any

    environment {
    //    NETLIFY_SITE_ID = 'a22cc808-5d50-4b4c-946d-8938b7a2e6eb'
    //    NETLIFY_AUTH_TOKEN = credentials('netlify-token')
       REACT_APP_VERSION = "1.0.$BUILD_ID"
       AWS_DEFAULT_REGION = 'us-east-1'
    }

    stages {       
        // required for first time build
        // stage('Docker') {
        //     steps {
        //         sh 'docker build -t my-playwright .'
        //     }
        // }

        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli:2.28.11'
                    reuseNode true
                    args  "--entrypoint=''" // --entrypoint=''
                }
            }

            //environment {
                //AWS_S3_BUCKET = 'learn-jenkins-202508170000'
            //}

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-jenkins-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                    aws --version
                    yum install jq -y
                    #aws s3 sync build s3://$AWS_S3_BUCKET
                    LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                    echo $LATEST_TD_REVISION
                    aws ecs update-service --cluster LearnJenkinsApp-Cluster-Pro \\
                        --service LearnJenkinsApp-Servuce-Prod --task-definition LearnJenkinsApp-TaskDefinition-Prod:$LATEST_TD_REVISION
                    '''
                }
                
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

        

        // stage('Tests') {
        //     parallel {
        //         stage('Unit tests') {
        //             agent {
        //                 docker {
        //                     image 'node:22-alpine3.21'
        //                     reuseNode true
        //                 }
        //             }
        //             steps {
        //                 echo "Test stage"
        //                 sh '''
        //                     test -f build/index.html
        //                     npm test
        //                 '''
        //             }

        //             post {
        //                 always {
        //                     junit 'jest-results/junit.xml'
        //                 }
        //             }

        //         }

        //         stage('E2E') {
        //             agent {
        //                 docker {
        //                     image 'my-playwright'
        //                     reuseNode true
        //                 }
        //             }
        //             steps {
        //                 echo "Test stage"
        //                 sh '''
        //                     serve -s build &
        //                     # wait 10 seconds for the server to start:
        //                     sleep 10
        //                     npx playwright test --reporter=html
        //                 '''
        //             }

        //             post {
        //                 always {
        //                     publishHTML([
        //                         allowMissing: false, 
        //                         alwaysLinkToLastBuild: false, 
        //                         icon: '', 
        //                         keepAll: false, 
        //                         reportDir: 'playwright-report', 
        //                         reportFiles: 'index.html', 
        //                         reportName: 'Playwright Local Report', 
        //                         reportTitles: '', 
        //                         useWrapperFileDirectly: true
        //                     ])
        //                 }
        //             }
        //         }
        //     }
        // }

        // stage('Deploy Staging') {
        //     agent {
        //         docker {
        //             image 'my-playwright'
        //             reuseNode true
        //         }
        //     }

        //     environment {
        //          CI_ENVIRONMENT_URL = "STAGING_URL_TO_BE_SET"
        //     }

        //     steps {
        //         echo "Test stage"
        //         sh '''
        //             netlify --version
                    
        //             echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"                 
        //             netlify status
                                        
        //             netlify deploy --dir=build --no-build --json > deploy-output.json
                                             
        //             CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deploy-output.json)                     
                    
        //             npx playwright test --reporter=html
        //         '''
        //     }

        //     post {
        //         always {
        //             publishHTML([
        //                 allowMissing: false, 
        //                 alwaysLinkToLastBuild: false, 
        //                 icon: '', 
        //                 keepAll: false, 
        //                 reportDir: 'playwright-report', 
        //                 reportFiles: 'index.html', 
        //                 reportName: 'Staging E2E Report', 
        //                 reportTitles: '', 
        //                 useWrapperFileDirectly: true
        //             ])
        //         }
        //     }
        // }

        // stage('Deply Prod') {
        //     agent {
        //         // this image already includes NodeJS:
        //         docker {
        //             image 'my-playwright'
        //             reuseNode true
        //         }
        //     }

        //     environment {
        //          CI_ENVIRONMENT_URL = 'https://ornate-creponne-3d63e0.netlify.app'
        //     }

        //     steps {
        //         echo "Test stage"
        //         sh '''
        //             node --version                
        //             netlify --version
        //             echo "Deploying to production. Project ID: $NETLIFY_SITE_ID" 
        //             netlify status
        //             netlify deploy --dir=build --prod --no-build
        //             # Adding some sleep time to ensure the deployment is complete
        //             npx playwright test --reporter=html
        //         '''
        //     }

        //     post {
        //         always {
        //             publishHTML([
        //                 allowMissing: false, 
        //                 alwaysLinkToLastBuild: false, 
        //                 icon: '', 
        //                 keepAll: false, 
        //                 reportDir: 'playwright-report', 
        //                 reportFiles: 'index.html', 
        //                 reportName: 'Prod E2E Report', 
        //                 reportTitles: '', 
        //                 useWrapperFileDirectly: true
        //             ])
        //         }
        //     }
        // }

        


    }


    

}
