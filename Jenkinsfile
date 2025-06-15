pipeline {
    agent any

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
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
            
        }

        stage('Test') {
            steps {
                echo "Test stage"
                sh 'test -f build/index.html'
                sh 'npm test'
            }
        }
    }
}
