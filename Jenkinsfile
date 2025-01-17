#!/usr/bin/env groovy

pipeline {
    agent any

    options {
        ansiColor('xterm')
        skipDefaultCheckout()
    }

    environment {
        AWS_ACCOUNT_ID="484301408782"
        AWS_DEFAULT_REGION="us-east-1" 
        IMAGE_REPO_NAME="hello-world"
        IMAGE_TAG="${BUILD_NUMBER}"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    }
   
    stages {

        stage ('Get latest code') {
            steps {
                checkout scm
            }
        }
        
        stage('Log into AWS ECR') {
            steps {
                script {
                    sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                }
            }
        }
        
        stage('Building image') {
            steps {
                script {
                    sh "docker build --no-cache -t ${IMAGE_REPO_NAME}:${IMAGE_TAG} ."
                }
            }
        }
   
        stage('Pushing to ECR') {
            steps {  
                script {
                    sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"
                    sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
                    sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:latest"
                    sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:latest"
                }
            }
        }

        stage('Deploying to EKS') {
           steps {
               script {
                   sh "kubectl apply -f deployment.yml"
               }
           }
        }
    }
}
