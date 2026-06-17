pipeline {
    agent any
 
    environment {
        AWS_DEFAULT_REGION = 'eu-north-1'
        S3_BUCKET = 'front-demo-pro'
        CLOUDFRONT_DISTRIBUTION_ID = 'E9OL989T16KEQ'
    }
 
    stages {
        stage('Clone Frontend Repo') {
            steps {
                git branch: 'main', 
                credentialsId: 'github-creds', 
                url: 'https://github.com/kavink112/jenkins-frontend.git'
            }
        }
 
        stage('Install & Build') {
            steps {
                sh '''
                
                rm -rf node_modules package-lock.json
 
                npm install --legacy-peer-deps || true
 
                npm run build
                
                '''
            }
        }
 
        stage('Deploy to S3') {
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-creds',  secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                    aws s3 sync ./build s3://$S3_BUCKET --delete
                    '''
                }
            }
        }
 
        stage('Invalidate CloudFront') {
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-creds', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                    aws cloudfront create-invalidation \
                      --distribution-id $CLOUDFRONT_DISTRIBUTION_ID \
                      --paths "/*"
                    '''
                }
            }
        }
    }
}
