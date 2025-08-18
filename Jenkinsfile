pipeline {
    agent any

    environment {
       REACT_APP_VERSION = "1.0.$BUILD_ID"
       APP_NAME = 'learnjenkinsapp'
       AWS_DEFAULT_REGION = 'us-east-1'
       AWS_DOCKER_REGISTRY = '653290752755.dkr.ecr.us-east-1.amazonaws.com'
       AWS_ECS_CLUSTER = 'LearnJenkinsApp-Cluster-Pro'
       AWS_ECS_SERVICE_PROD = 'LearnJenkinsApp-Servuce-Prod'
       AWS_ECS_TD_PROD = 'LearnJenkinsApp-TaskDefinition-Prod'
    }

    stages {       
        // required for first time build
        // stage('Docker') {
        //     steps {
        //         sh 'docker build -t my-playwright .'
        //     }
        // }


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

        stage('Build Docker Image') {

            agent {
                docker {
                    image 'my-aws-cli'
                    reuseNode true
                    args  "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''" // --entrypoint=''
                }
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-jenkins-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        #uname -a
                        #amazon-linux-extras install docker
                        #apt-get install docker -y
                        #yum install docker -y
                        docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION .
                        aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
                        docker push $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION
                    '''
                }
            }
        }

        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'my-aws-cli'
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
                    sed -i "s/#APP_VERSION#/$REACT_APP_VERSION/g" aws/task-definition-prod.json
                    #yum install jq -y
                    #aws s3 sync build s3://$AWS_S3_BUCKET
                    LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                    #echo $LATEST_TD_REVISION
                    aws ecs update-service --cluster $AWS_ECS_CLUSTER \\
                        --service $AWS_ECS_SERVICE_PROD --task-definition $AWS_ECS_TD_PROD:$LATEST_TD_REVISION
                    aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER \\
                        --services $AWS_ECS_SERVICE_PROD
                    '''
                }
                
            }
        }

    }


    

}
