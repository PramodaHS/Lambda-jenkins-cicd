pipeline {
    agent any

    environment {
        AWS_CREDENTIALS = credentials('aws-credentials')
        S3_BUCKET = 'pramod-lambda-deployments'
        FUNCTION_NAME = 'myLambdaFunction'
        AWS_REGION = 'ap-south-1'
        SLACK_WEBHOOK_URL = credentials('ee5739a3-4b28-4c40-8baf-dd3a794ddcf3') 
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    git branch: 'master', url: 'https://github.com/PramodaHS/Lambda-jenkins-cicd.git'
                }
            }
        }

        stage('Package Lambda') {
            steps {
                sh 'zip lambda-package.zip lambda_function.py'
            }
        }

        stage('Upload to S3') {
            steps {
                sh 'aws s3 cp lambda-package.zip s3://$S3_BUCKET/lambda-package.zip --region $AWS_REGION'
            }
        }

        stage('Deploy to Lambda') {
            steps {
                sh '''
                aws lambda update-function-code \
                --function-name $FUNCTION_NAME \
                --s3-bucket $S3_BUCKET \
                --s3-key lambda-package.zip \
                --region $AWS_REGION
                '''
            }
        }
    }

    post {
        always {
            sh 'rm -f lambda-package.zip'
        }
        success {
            script {
                slackSend (
                    color: 'good',
                    message: "✅ *Deployment Successful!* :rocket:\nBranch: `us-4010`\nLambda Function: `$FUNCTION_NAME`",
                    channel: '#deployments'
                )
            }
        }
        failure {
            script {
                slackSend (
                    color: 'danger',
                    message: "❌ *Deployment Failed!* :x:\nCheck Jenkins logs for more details.",
                    channel: '#deployments'
                )
            }
        }
    }
}
