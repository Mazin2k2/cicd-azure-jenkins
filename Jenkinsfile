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
        KUBE_CONFIG = credentials('aks-kubeconfig')  // Jenkins secret containing your AKS kubeconfig
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

        stage('Deploy to AKS') {
            steps {
                script {
                    // Replace placeholders in the web-app.yaml with the actual values
                    sh """
                        sed -i 's|${ACR_URL}/${IMAGE_NAME}:.*|${ACR_URL}/${IMAGE_NAME}:${IMAGE_TAG}|g' web-app.yaml
                    """

                    // Deploy the web-app.yaml if the deployment does not exist
                    withCredentials([file(credentialsId: 'aks-kubeconfig', variable: 'KUBECONFIG')]) {
                        sh '''
                            export KUBECONFIG=${KUBECONFIG}

                            # Apply the Kubernetes manifest (this will create the deployment if it doesn't exist)
                            kubectl apply -f web-app.yaml
                        '''
                    }

                    // Update the deployment image in AKS
                    withCredentials([file(credentialsId: 'aks-kubeconfig', variable: 'KUBECONFIG')]) {
                        sh '''
                            export KUBECONFIG=${KUBECONFIG}

                            # Update the image of the deployment
                            kubectl set image deployment/python-web-app python-web-app=${ACR_URL}/${IMAGE_NAME}:${IMAGE_TAG} --record
                        '''
                    }
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
            echo 'Docker image successfully built, pushed to ACR, and deployed to AKS!'
        }
        failure {
            echo 'There was an error in the pipeline!'
        }
    }
}
