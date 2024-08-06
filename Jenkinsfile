pipeline {
    agent any
    environment {
        AWS_REGION = 'your-aws-region' // Replace with your AWS region
    }
    stages {
        stage('Build') {
            steps {
                script {
                    // Run Gradle build
                    sh './gradlew clean build'
                }
            }
        }
        stage('Package') {
            steps {
                script {
                    // Create a deployment package (e.g., a zip file with the JAR)
                    sh 'mkdir -p build/deploy'
                    sh 'cp build/libs/*.jar build/deploy/function.jar'
                    sh 'cd build/deploy && zip -r function.zip .'
                }
            }
        }
        stage('Deploy to AWS Lambda') {
            steps {
                script {
                    def branchName = env.BRANCH_NAME
                    def lambdaFunctionName

                    switch (branchName) {
                        case 'dev':
                            lambdaFunctionName = 'your-dev-lambda-function-name'
                            break
                        case 'QA':
                            lambdaFunctionName = 'your-qa-lambda-function-name'
                            break
                        case 'ACC':
                            lambdaFunctionName = 'your-acc-lambda-function-name'
                            break
                        default:
                            error "No Lambda function configured for branch ${branchName}"
                    }

                    // Deploy to AWS Lambda
                    sh """
                        aws lambda update-function-code --function-name ${lambdaFunctionName} --zip-file fileb://build/deploy/function.zip --region ${AWS_REGION}
                    """
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}

