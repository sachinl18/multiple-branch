pipeline {
    agent any

    parameters {
        choice(name: 'BRANCH_NAME', choices: ['dev', 'qa', 'acc'], description: 'Select the branch for deployment')
    }

    environment {
        AWS_REGION = 'us-west-2'
        LAMBDA_FUNCTION_NAME_DEV = 'myLambdaDev'
        LAMBDA_FUNCTION_NAME_QA = 'myLambdaQA'
        LAMBDA_FUNCTION_NAME_ACC = 'myLambdaACC'
        JAR_FILE = 'app/build/libs/jb-hello-world-0.1.0.jar'
        S3_BUCKET = 'sachin-s3-lambda-bucket'
    }

    stages {
        stage('Build') {
            steps {
                sh './gradlew clean build'
            }
        }

        stage('Upload to S3') {
            steps {
                withAWS(credentials: 'aws-cred', region: "${env.AWS_REGION}") {
                    sh """
                    aws s3 cp ${env.WORKSPACE}/${env.JAR_FILE} s3://${env.S3_BUCKET}/${env.JAR_FILE}
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def lambdaFunctionName

                    switch (params.BRANCH_NAME) {
                        case 'dev':
                            lambdaFunctionName = env.LAMBDA_FUNCTION_NAME_DEV
                            break
                        case 'qa':
                            lambdaFunctionName = env.LAMBDA_FUNCTION_NAME_QA
                            break
                        case 'acc':
                            lambdaFunctionName = env.LAMBDA_FUNCTION_NAME_ACC
                            break
                        default:
                            error "Branch ${params.BRANCH_NAME} is not recognized for deployment"
                    }

                    withAWS(credentials: 'aws-cred', region: "${env.AWS_REGION}") {
                        sh """
                        aws lambda update-function-code \
                            --region ${env.AWS_REGION} \
                            --function-name ${lambdaFunctionName} \
                            --s3-bucket ${env.S3_BUCKET} \
                            --s3-key ${env.JAR_FILE}
                        """
                    }
                }
            }
        }
    }
}
