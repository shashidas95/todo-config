pipeline {
    agent any

    parameters {
        string(name: 'IMAGE_TAG', description: 'Enter the image tag')
    }

    environment {
        GIT_REPO = "https://github.com/shashidas95/todo-config.git"
        DOCKERHUB_USERNAME = "shashidas" // Docker Hub username
        IMAGE_BE = "todo-be"
        IMAGE_FE = "todo-fe"
        CONFIG_PROJECT_NAME = "todo-config"
        GOOGLE_CHAT_WEBHOOK = credentials('google_chat_webhook')
        DOCKER_SERVER = '3.110.119.81'
        SSH_KEY = credentials('docker-shashi-key') // SSH key stored in Jenkins credentials
    }

    stages {
        stage("CLEANUP WORKSPACE") {
            steps {
                cleanWs()
            }
        }

        stage("CHECKOUT GIT REPO") {
            steps {
                git branch: 'main', url: "${GIT_REPO}"
            }
        }

        stage("SEND GOOGLE CHAT NOTIFICATION") {
            steps {
                script {
                    def message = "{\"text\": \"Starting deployment with new image tag: ${IMAGE_TAG}\"}"
                    sh """
                        curl -X POST -H 'Content-Type: application/json' -d '${message}' ${GOOGLE_CHAT_WEBHOOK}
                    """
                }
            }
        }

      stage("SSH TO DOCKER SERVER AND UPDATE IMAGES") {
    steps {
        script {
            echo "Starting SSH connection to the Docker server..."
            sshagent(credentials: ['docker_server_credentials']) {
                sh """
                    # Debugging output to ensure we are in the correct environment
                    echo "Checking SSH connection..."
                    ssh -o StrictHostKeyChecking=no ubuntu@${DOCKER_SERVER} 'echo "SSH connection successful"'

                    # Navigate to project directory and pull latest changes
                    echo "Navigating to project directory..."
                    ssh ubuntu@${DOCKER_SERVER} 'cd ${CONFIG_PROJECT_NAME} && git pull origin main'

                    # Update the image tags in Kubernetes files and docker-compose.yaml
                    echo "Updating image tags in Kubernetes and docker-compose.yaml..."
                    ssh ubuntu@${DOCKER_SERVER} """
                    sed -i 's#image: ${DOCKERHUB_USERNAME}/${IMAGE_BE}:.*#image: ${DOCKERHUB_USERNAME}/${IMAGE_BE}:${IMAGE_TAG}#' ./k8s/backend-deployment.yaml
                    sed -i 's#image: ${DOCKERHUB_USERNAME}/${IMAGE_FE}:.*#image: ${DOCKERHUB_USERNAME}/${IMAGE_FE}:${IMAGE_TAG}#' ./k8s/frontend-deployment.yaml
                    sed -i 's#image: ${DOCKERHUB_USERNAME}/${IMAGE_BE}:.*#image: ${DOCKERHUB_USERNAME}/${IMAGE_BE}:${IMAGE_TAG}#' docker-compose.yaml
                    sed -i 's#image: ${DOCKERHUB_USERNAME}/${IMAGE_FE}:.*#image: ${DOCKERHUB_USERNAME}/${IMAGE_FE}:${IMAGE_TAG}#' docker-compose.yaml
                    """

                    # Commit and push changes to git
                    echo "Committing and pushing changes to GitHub..."
                    ssh ubuntu@${DOCKER_SERVER} 'cd ${CONFIG_PROJECT_NAME} && git add . && git commit -m "Updated image tags with ${IMAGE_TAG}" || echo "No changes to commit" && git push origin main'

                    # Restart Docker Compose services
                    echo "Restarting Docker Compose services..."
                    ssh ubuntu@${DOCKER_SERVER} 'cd ${CONFIG_PROJECT_NAME} && docker-compose down && docker-compose up -d --no-cache'
                """
            }
        }
    }
}

    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}
