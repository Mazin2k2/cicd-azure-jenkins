pipeline {
    agent any

    environment {
        ACR_NAME = 'testacr0909'  // Your Azure Container Registry name
        ACR_URL = "${ACR_NAME}.azurecr.io"
        IMAGE_NAME = 'pyimg'  // The Docker image name
        IMAGE_TAG = "${env.BUILD_ID}"  // Use Jenkins build ID for unique tagging
        ACR_USERNAME = 'testacr0909'  // ACR username (same as ACR registry name)
        ACR_PASSWORD = credentials('acr-access-key')  // Jenkins secret with your ACR access key
        GITHUB_REPO = 'https://github.com/Mazin2k2/cicd-azure-jenkins.git'  // Your GitHub repository
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Checkout the repository code from GitHub
                git branch: 'main', url: "${GITHUB_REPO}"
            }
        }

        stage('Login to ACR') {
            steps {
                script {
                    // Login to Azure Container Registry using ACR access key
                    sh '''
                        docker login ${ACR_URL} -u ${ACR_USERNAME} -p ${ACR_PASSWORD}
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image and tag it with the ACR URL and unique image tag
                    sh """
                        docker build -t ${ACR_URL}/${IMAGE_NAME}:${IMAGE_TAG} .
                    """
                }
            }
        }

        stage('Push Docker Image to ACR') {
            steps {
                script {
                    // Push the Docker image to Azure Container Registry
                    sh """
                        docker push ${ACR_URL}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Clean up Docker Images') {
            steps {
                script {
                    // Clean up local Docker images to save space
                    sh """
                        docker rmi ${ACR_URL}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Docker image successfully built and pushed to ACR!'
        }
        failure {
            echo 'There was an error in the pipeline!'
        }
    }
}
