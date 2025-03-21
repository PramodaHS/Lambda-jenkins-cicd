pipeline {
    agent any

    environment {
        AWS_CREDENTIALS = credentials('aws-credentials')
        S3_BUCKET = 'pramod-lambda-deployments'
        FUNCTION_NAME = 'myLambdaFunction'
        AWS_REGION = 'ap-south-1'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'US-4010', url: 'https://github.com/PramodaHS/Lambda-jenkins-cicd.git'
                git branch: 'master', url: 'https://github.com/PramodaHS/Lambda-jenkins-cicd.git'
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
            slackSend channel: '#jenkins', message: " *Deployment Successful!*\n\n*Job:* ${JOB_NAME} \n*Branch:* ${GIT_BRANCH} \n*Build Number:* ${BUILD_NUMBER} \n*View Job:* ${BUILD_URL}"
        }
        failure {
            slackSend channel: '#jenkins', message: " *Deployment Failed!*\n\n*Job:* ${JOB_NAME} \n*Branch:* ${GIT_BRANCH} \n*Build Number:* ${BUILD_NUMBER} \n*View Job:* ${BUILD_URL}"
        }
    }
}

