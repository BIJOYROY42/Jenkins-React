pipeline {
    agent any
    environment{
        AWS_DEFAULT_REGION = 'us-east-2'
    }
    stages {
        // stage('Build') {
        //     agent {
        //         docker {
        //             image 'node:22-alpine'
        //             // for the same docker image, reuse
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         sh '''
        //             # list all files
        //             ls -la
        //             node --version
        //             npm --version
        //             # npm install
        //             npm ci
        //             npm run build
        //             ls -la
        //         '''
        //     }
        // }

        // stage('Test') {
        //     agent {
        //         docker {
        //             image 'node:22-alpine'
        //             reuseNode true
        //         }
        //     }

        //     steps {
        //         sh '''
        //             test -f build/index.html
        //             npm test
        //         '''
        //     }
        // }
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
                        aws ecs update-service --cluster Temp-Cluster-Prod --service Temp-Service-Prod --task-definition Temp-TaskDefinition-Prod:4
                    '''
                }
            }
        }
    }
}
