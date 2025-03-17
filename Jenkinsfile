pipeline {
    agent any

environment {

        AWS_DOCKER_REGISTRY = '612634926349.dkr.ecr.us-east-2.amazonaws.com'
        // your ECR repository name
        APP_NAME = 'tempapp'
        REACT_APP_VERSION = '1.0.1'
        AWS_DEFAULT_REGION = 'us-east-2'
        AWS_ECS_CLUSTER = 'tempApp-Cluster-Prod'
        AWS_ECS_SERVICE_PROD = 'tempApp-Service-Prod'
        AWS_ECS_TD_PROD = 'tempApp--TaskDefinition-Prod'

    }
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
                    args '-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=""'
                }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        amazon-linux-extras install docker
                        docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION .
                        aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
                        docker push $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION
                    '''
                }
            }
        }

        stage('Deploy to AWS'){
            agent{
                docker{
                    image 'amazon/aws-cli'
                    reuseNode true
                    args '--entrypoint=""'
                }
            }

            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    // some block
                    sh '''
                        aws --version
                        aws ecs register-task-definition --cli-input-json file://aws/task-definition.json
                        aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE_PROD --task-definition $AWS_ECS_TD_PROD:6
                    '''
                }
            }
        }
    }
}
