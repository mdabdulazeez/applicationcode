pipeline {
    agent any

    environment {
        IMAGE_NAME = "backend-app"
        IMAGE_TAG = "v${BUILD_NUMBER}"
        REGISTRY_URL = "localhost:5000"
    }

    stages {
        stage('Build and Push Docker Image') {
            steps {
                script {
                    // Build the image
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                    
                    // Push to registry
                    docker.withRegistry("http://${REGISTRY_URL}", 'nexus-docker-credentials') {
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Update Kubernetes Manifest') {
            environment {
                GITHUB_TOKEN = credentials('github-token')
            }
            steps {
                sh """
                    # Clone manifests repo
                    git clone https://github.com/mdabdulazeez/manifests.git
                    cd manifests
                    
                    # Update image tag
                    sed -i "s|image: .*|image: ${REGISTRY_URL}/${IMAGE_NAME}:${IMAGE_TAG}|" deployment.yaml
                    
                    # Commit and push
                    git config user.name "jenkins"
                    git config user.email "jenkins@ci.local"
                    git add deployment.yaml
                    git commit -m "Update image to ${REGISTRY_URL}/${IMAGE_NAME}:${IMAGE_TAG}"
                    git push https://${GITHUB_TOKEN}@github.com/mdabdulazeez/manifests.git main
                """
            }
        }
    }
}
