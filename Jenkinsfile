pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE_NAME = 'html-app-image'
        DOCKER_REGISTRY = 'akshaykomath/akshay_ks:tagname'
        REMOTE_SERVER = '3.15.177.51'
        REMOTE_USER = 'ubuntu'
        REMOTE_DIRECTORY = '/tmp/'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the code from Git repository
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh "docker build -t ${DOCKER_IMAGE_NAME} ."
                }
            }
        }
        
        stage('Save Docker Image') {
            steps {
                script {
                    // Save Docker image to a tar file
                    sh "docker save -o ${DOCKER_IMAGE_NAME}.tar ${DOCKER_IMAGE_NAME}"
                }
            }
        }
        
        stage('Publish Docker Image') {
            steps {
                script {
                    // Optionally, push Docker image to a Docker registry
                    // sh "docker tag ${DOCKER_IMAGE_NAME} ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}"
                    // sh "docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}"
                }
            }
        }
        
        stage('Transfer to Remote Server') {
            steps {
                script {
                    // Transfer the Docker image to the remote server
                    sshPublisher(publishers: [
                        sshPublisherDesc(
                            configName: 'web-server',
                            transfers: [
                                sshTransfer(
                                    sourceFiles: "${DOCKER_IMAGE_NAME}.tar",
                                    remoteDirectory: "${REMOTE_DIRECTORY}",
                                    removePrefix: '',
                                    execCommand: ''
                                )
                            ],
                            usePromotionTimestamp: false,
                            useWorkspaceInPromotion: false,
                            verbose: false
                        )
                    ])
                }
            }
        }
        
        stage('Deploy on Remote Server') {
            steps {
                script {
                    // Execute commands on the remote server to load and run the Docker container
                    sshCommand remote: [name: 'web-server'], command: '''
                        docker load -i ${REMOTE_DIRECTORY}/${DOCKER_IMAGE_NAME}.tar
                        docker stop html-app-container || true
                        docker rm html-app-container || true
                        docker run -d -p 8080:80 --name html-app-container ${DOCKER_IMAGE_NAME}
                    '''
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
