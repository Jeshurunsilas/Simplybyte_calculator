
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'jeshurunsilas/simplybyte-calculator'
        AWS_REGION = 'ap-southeast-2'
        EC2_INSTANCE_ID = 'i-042d94fa527aaf01e'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Source code downloaded from GitHub.'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building SimplyByte Calculator image...'
                bat 'docker build -t %IMAGE_NAME%:latest .'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    bat '''
                    powershell -NoProfile -NonInteractive -Command "$env:DOCKER_TOKEN | docker login -u $env:DOCKER_USER --password-stdin"
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                echo 'Uploading image to Docker Hub...'
                bat 'docker push %IMAGE_NAME%:latest'
            }
        }

        stage('Verify AWS Credentials') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-deploy-credentials',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    bat '''
                    set AWS_DEFAULT_REGION=%AWS_REGION%
                    aws sts get-caller-identity
                    '''
                }
            }
        }

        stage('Deploy to Production EC2') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-deploy-credentials',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    powershell '''
                    $env:AWS_DEFAULT_REGION = "ap-southeast-2"

                    $instance = "i-042d94fa527aaf01e"

                    $parameters = 'commands=["sudo docker pull jeshurunsilas/simplybyte-calculator:latest","sudo docker rm -f simplybyte-calculator || true","sudo docker run -d --restart unless-stopped --name simplybyte-calculator -p 5000:5000 jeshurunsilas/simplybyte-calculator:latest"]'

                    Write-Host "Sending deployment command to AWS..."

                    $commandId = aws ssm send-command `
                        --instance-ids $instance `
                        --document-name "AWS-RunShellScript" `
                        --parameters $parameters `
                        --region $env:AWS_DEFAULT_REGION `
                        --query "Command.CommandId" `
                        --output text

                    if ($LASTEXITCODE -ne 0) {
                        throw "Failed to send AWS deployment command."
                    }

                    Write-Host "Deployment command ID: $commandId"

                    aws ssm wait command-executed `
                        --command-id $commandId `
                        --instance-id $instance `
                        --region $env:AWS_DEFAULT_REGION

                    if ($LASTEXITCODE -ne 0) {
                        throw "EC2 deployment failed."
                    }

                    Write-Host "Production EC2 deployment completed!"
                    '''
                }
            }
        }
    }

    post {
        always {
            bat 'docker logout'
        }

        success {
            echo 'SUCCESS: Docker image pushed and production EC2 deployment completed!'
        }

        failure {
            echo 'FAILED: Check the Jenkins console output for the error.'
        }
    }
}
