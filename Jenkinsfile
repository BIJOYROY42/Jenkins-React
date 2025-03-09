pipeline {
    agent any
    // In jenkins, test docker first
    // Add jenkinsfile in react app, run hello world template
    // Add Build and Test stage 
    environment{
        NETLIFY_SITE_ID = 'cc8030f9-3a86-4cf4-b83f-cfa3cc7b78ce'
        NETLIFY_AUTH_TOKEN = credentials('Jenkins-React')
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:22-alpine'
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
                    image 'node:22-alpine'
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
        stage('Deploy') {
            agent {
                docker {
                    image 'node:22-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    # deploy to build folder
                    node_modules/.bin/netlify deploy --prod --dir=build
                '''
            }
        }

    }
}
