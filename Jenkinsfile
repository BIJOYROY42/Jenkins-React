pipeline {
    agent any
    // In jenkins, test docker first
    // Add jenkinsfile in react app, run hello world template
    // Add Build and Test stage 
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    // alow jenkins to reuse the same agent for entire pipline
                    reuseNode true
                }
            }
            steps {
                sh '''
                    # list all files
                    ls -la
                    node --version
                    npm --version
                    # npm install
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Test') {
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
        }
    }
}
