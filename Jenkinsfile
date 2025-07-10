pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '098588167308'    
        ECR_REPO_NAME = 'jenkins'           
        IMAGE_TAG = 'v2'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        FULL_IMAGE_NAME = "${ECR_REGISTRY}/${ECR_REPO_NAME}:${IMAGE_TAG}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t $ECR_REPO_NAME:$IMAGE_TAG .'
                }
            }
        }

        stage('Tag Image') {
            steps {
                script {
                    sh 'docker tag $ECR_REPO_NAME:$IMAGE_TAG $FULL_IMAGE_NAME'
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                script {
                    sh '''
                        aws ecr get-login-password --region $AWS_REGION | \
                        docker login --username AWS --password-stdin $ECR_REGISTRY
                    '''
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    sh 'docker push $FULL_IMAGE_NAME'
                }
            }
        }

        stage('Login to AWS ECR for Pull') {
            steps {
                script {
                    sh '''
                        aws ecr get-login-password --region $AWS_REGION | \
                        docker login --username AWS --password-stdin $ECR_REGISTRY
                    '''
                }
            }
        }

        stage('Pull from DockerHub') {
            steps {
                script {
                    sh "docker pull ${FULL_IMAGE_NAME}"
                }
            }
        }

        stage('Create Pods form Build Image') {
            steps {
                script {
                    sh "docker run -itd -p 5000:80 --name jenkins ${FULL_IMAGE_NAME}"
                }
            }
        }
    }
    post {
        always {
            mail to: 'ashoksulthan21@gmail.com',
                 subject: "Jenkins Build Notification: ${currentBuild.fullDisplayName}",
                 body: """\
                 Build Status: ${currentBuild.currentResult}
                 Project: ${env.JOB_NAME}
                 Build Number: ${env.BUILD_NUMBER}
                 Build URL: ${env.BUILD_URL}
                 """
        }
    }
}
