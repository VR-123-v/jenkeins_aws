pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = "014869289060"                    // Your AWS Account ID
        AWS_DEFAULT_REGION = "us-east-1"                  // Region
        IMAGE_REPO_NAME = "devopsvimal"                  // Your ECR repo
        IMAGE_TAG = "latest"                                 // Tag for this build
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    }

    stages {
        stage('Login to AWS ECR') {
            steps {
                script {
                    sh """
                    aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | \
                    docker login --username AWS --password-stdin ${REPOSITORY_URI}
                    """
                }
            }
        }

        stage('Clone Git Repo') {
            steps {
                checkout([$class: 'GitSCM',
                          branches: [[name: '*/main']],
                          userRemoteConfigs: [[url: 'https://github.com/VR-123-v/jenkeins_aws.git']]
                ])
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${IMAGE_REPO_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                script {
                    sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:${IMAGE_TAG}"
                    sh "docker push ${REPOSITORY_URI}:${IMAGE_TAG}"
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning up local Docker images..."
            sh "docker image prune -f || true"
        }
    }
}
