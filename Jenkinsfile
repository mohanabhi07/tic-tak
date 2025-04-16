pipeline {
    agent any
    
    environment {
        // Configure these variables for your project
        DOCKER_HUB_REPO = "mohanabhi/tic-tak"
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKERFILE_PATH = "." // Path to directory containing Dockerfile
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Check out code from GitHub repository
                checkout scm
                // Alternatively, specify your repository directly:
                // git branch: 'main', url: 'https://github.com/yourusername/your-repo.git'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                dir(DOCKERFILE_PATH) {
                    script {
                        // Build the Docker image
                        sh "docker build -t ${DOCKER_HUB_REPO}:${IMAGE_TAG} ."
                        // Also tag as latest
                        sh "docker tag ${DOCKER_HUB_REPO}:${IMAGE_TAG} ${DOCKER_HUB_REPO}:latest"
                    }
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    // Log in to Docker Hub
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', 
                                                     passwordVariable: 'DOCKER_PASSWORD', 
                                                     usernameVariable: 'DOCKER_USERNAME')]) {
                        sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
                        
                        // Push both tags
                        sh "docker push ${DOCKER_HUB_REPO}:${IMAGE_TAG}"
                        sh "docker push ${DOCKER_HUB_REPO}:latest"
                    }
                }
            }
        }
    }
    
    post {
        always {
            // Clean up local Docker images to save space
            sh "docker rmi ${DOCKER_HUB_REPO}:${IMAGE_TAG} || true"
            sh "docker rmi ${DOCKER_HUB_REPO}:latest || true"
            
            // Log out from Docker Hub
            sh "docker logout"
        }
        success {
            echo "Successfully built and pushed Docker image ${DOCKER_HUB_REPO}:${IMAGE_TAG}"
        }
        failure {
            echo "Failed to build and push Docker image"
        }
    }
}