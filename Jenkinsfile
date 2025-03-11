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
        stage('AWS'){
            agent{
                docker{
                    image 'amazon/aws-cli'
                    args '--entrypoint=""'
                }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    // some block
                    sh '''
                        aws --version
                        echo "Hello S3!" > index.html
                        aws s3 cp index.html s3://my-jenkins-20250308/index.html
                    '''
                }
            }
        }
        stage('Docker'){
            steps{
                sh 'docker build -t my-docker-image .'
            }
        }
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
                    // image 'node:22-alpine'
                    image 'my-docker-image'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    # npm install netlify-cli
                    # node_modules/.bin/netlify --version
                    # echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    # node_modules/.bin/netlify status
                    # # deploy to build folder
                    # node_modules/.bin/netlify deploy --prod --dir=build

                    ###### custom docker image
                    netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --prod --dir=build
                '''
            }
        }
    }
}
