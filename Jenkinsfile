pipeline {
    agent any

    stages {
        
        stage('Build') {
            agent {
                docker {
                    image 'node:22-alpine'
                    // for the same docker image, reuse
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

        stage('Build My Docker Image'){

            agent{
                docker{
                    image 'amazon/aws-cli'
                    reuseNode true
                    // set up sock to communicate with docker daemon
                    args '-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=""'
                }
            }
            steps{
                // In amazon/aws-cli, there is no docker installed, 
                // so we need to install docker to run "docker build" command
                sh '''
                    amazon-linux-extras install docker
                    docker build -t my-docker-image .
                '''
            }
        }
    }
}
