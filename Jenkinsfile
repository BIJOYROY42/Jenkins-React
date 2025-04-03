pipeline {
    agent any

    environment {
        S3_BUCKET = 'myfirstawsbucket422025'  
        AWS_DEFAULT_REGION = 'us-east-2'
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:22-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm ci
                    npm run build
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
                '''
            }
        }

        stage('Deploy to S3') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args '-u root --entrypoint=""'
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-temp-1', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        aws s3 ls
                        echo "Hello S3" > index.html
                        aws s3 cp index.html s3://myfirstawsbucket422025/index.html
                        # Create S3 bucket if it doesn't exist
                        aws s3api create-bucket --bucket $S3_BUCKET --region $AWS_DEFAULT_REGION --create-bucket-configuration LocationConstraint=$AWS_DEFAULT_REGION || true
                        
                        # Enable static website hosting
                        aws s3api put-bucket-website --bucket $S3_BUCKET --website-configuration '{"IndexDocument":{"Suffix":"index.html"},"ErrorDocument":{"Key":"index.html"}}'
                        
                        # Configure bucket policy for public access
                        aws s3api put-bucket-policy --bucket $S3_BUCKET --policy '{
                            "Version": "2012-10-17",
                            "Statement": [
                                {
                                    "Sid": "PublicReadGetObject",
                                    "Effect": "Allow",
                                    "Principal": "*",
                                    "Action": "s3:GetObject",
                                    "Resource": "arn:aws:s3:::'$S3_BUCKET'/*"
                                }
                            ]
                        }'
                        
                        # Upload build files to S3
                        aws s3 sync build/ s3://$S3_BUCKET/ --delete
                        
                        # Print the website URL
                        echo "Website URL: http://$S3_BUCKET.s3-website.$AWS_DEFAULT_REGION.amazonaws.com"
                    '''
                }
            }
        }
    }
}