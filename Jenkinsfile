pipeline {
    agent any

    environment {
        ACR_NAME = 'testacr0909'
        ACR_URL = "${ACR_NAME}.azurecr.io"
        IMAGE_NAME = 'pyimg'
        IMAGE_TAG = "${env.BUILD_ID}"
        ACR_USERNAME = 'testacr0909'
        ACR_PASSWORD = credentials('acr-access-key')  // Jenkins secret containing your ACR password
        ACR_EMAIL = 'mazin.abdulkarimrelambda.onmicrosoft.com'  // Your ACR email
        GITHUB_REPO = 'https://github.com/Mazin2k2/cicd-azure-jenkins.git'
        KUBE_CONFIG = credentials('aks-kubeconfig')  // Jenkins secret containing your AKS kubeconfig
        HELM_CHART_PATH = 'helm/mypyapp'  // Path to your Helm chart
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: "${GITHUB_REPO}"
            }
        }

        stage('Login to ACR') {
            steps {
                script {
                    sh '''
                        docker login ${ACR_URL} -u ${ACR_USERNAME} -p ${ACR_PASSWORD}
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        docker build -t ${ACR_URL}/${IMAGE_NAME}:${IMAGE_TAG} .
                    """
                }
            }
        }

        stage('Push Docker Image to ACR') {
            steps {
                script {
                    sh """
                        docker push ${ACR_URL}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Create Docker Registry Secret') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'aks-kubeconfig', variable: 'KUBECONFIG')]) {
                        // Create the docker registry secret in AKS
                        sh """
                            export KUBECONFIG=${KUBECONFIG}

                            # Create or update the docker registry secret
                            kubectl create secret docker-registry regcred \
                            --docker-server=${ACR_URL} \
                            --docker-username=${ACR_USERNAME} \
                            --docker-password=${ACR_PASSWORD} \
                            --docker-email=${ACR_EMAIL} \
                            --dry-run=client -o yaml | kubectl apply -f -
                        """
                    }
                }
            }
        }

        stage('Deploy to AKS using Helm') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'aks-kubeconfig', variable: 'KUBECONFIG')]) {
                        sh '''
                            export KUBECONFIG=${KUBECONFIG}

                            # Deploy the Helm chart with the dynamic image and tag
                            helm upgrade --install python-web-app ${HELM_CHART_PATH} \
                            --set appimage=${ACR_URL}/${IMAGE_NAME} \
                            --set apptag=${IMAGE_TAG}
                        '''
                    }
                }
            }
        }

        stage('Clean up Docker Images') {
            steps {
                script {
                    sh """
                        docker rmi ${ACR_URL}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Docker image successfully built, pushed to ACR, secret created, and deployed using Helm to AKS!'
        }
        failure {
            echo 'There was an error in the pipeline!'
        }
    }
}
