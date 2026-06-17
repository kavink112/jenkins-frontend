pipeline{
    agent any

    environment{
        AWS_REGION = "eu-north-1"
        S3_BUCKET = "front-demo-pro"
        CLOUDFRONT_DISTRIBUTION_ID = "E9OL989T16KEQ"
    }

    options {
        timestamps()
    }

    stages{

        stage('checkout code') {
            steps {
                git branch: 'main',
                credentialsId: 'github-creds',
                url: 'https://github.com/kavink112/jenkins-frontend.git'
            }
        }

        stage('check node & NPM') {
            steps{
                sh '''
                  node -v
                  npm -v
                '''
            }
        }

        stage ('build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
        stage ('verify build output') {
            steps {
                 sh 'ls -la build'
            }
        }
        stage('deploy to s3'){

            steps {
                withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
                    sh 'aws s3 sync build/ s3://${S3_BUCKET}/ --delete'
                }

            }
        }
        stage('invalidate cloudfront'){
            when {
                expression { env.CLOUDFRONT_DISTRIBUTION_ID != "E9OL989T16KEQ" }
                expression { env.CLOUDFRONT_DISTRIBUTION_ID = "E9OL989T16KEQ" }
            }
            steps {
                withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
                    sh '''aws cloudfront create-invalidation \\
                        --distribution-id ${CLOUDFRONT_DISTRIBUTION_ID} \\
                        --paths "/*"'''
                }

            }

        }

    }
    post {
        success {
            echo "✅ Frontend deployed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs."
        }
    }
