
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'jeshurunsilas/simplybyte-calculator'
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

    }

    post {
        success {
            echo 'SUCCESS: Docker image built and pushed to Docker Hub!'
        }

        failure {
            echo 'FAILED: Check the Console Output for errors.'
        }
    }
}
