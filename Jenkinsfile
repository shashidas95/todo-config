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
        DOCKER_SERVER = '3.110.119.81' // Replace with your Docker server's IP or hostname
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
                    sshagent(credentials: ['docker_server_credentials']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${DOCKER_SERVER} 'bash -s' << 'EOF'
                        
                        # Navigate to project directory
                        cd ${CONFIG_PROJECT_NAME}

                        # Pull latest changes
                        git pull origin main

                        # Update backend and frontend image tags in Kubernetes files and docker-compose.yaml
                        sed -i 's#image: ${DOCKERHUB_USERNAME}/${IMAGE_BE}:.*#image: ${DOCKERHUB_USERNAME}/${IMAGE_BE}:${IMAGE_TAG}#' ./k8s/backend-deployment.yaml
                        sed -i 's#image: ${DOCKERHUB_USERNAME}/${IMAGE_FE}:.*#image: ${DOCKERHUB_USERNAME}/${IMAGE_FE}:${IMAGE_TAG}#' ./k8s/frontend-deployment.yaml
                        sed -i 's#image: ${DOCKERHUB_USERNAME}/${IMAGE_BE}:.*#image: ${DOCKERHUB_USERNAME}/${IMAGE_BE}:${IMAGE_TAG}#' docker-compose.yaml
                        sed -i 's#image: ${DOCKERHUB_USERNAME}/${IMAGE_FE}:.*#image: ${DOCKERHUB_USERNAME}/${IMAGE_FE}:${IMAGE_TAG}#' docker-compose.yaml

                        # Commit and push the changes if any
                        git add ./k8s/backend-deployment.yaml ./k8s/frontend-deployment.yaml docker-compose.yaml
                        git commit -m 'Updated deployment files with new image tag ${IMAGE_TAG}' || echo "No changes to commit"
                        git push origin main

                        # Restart Docker Compose services with no cache
                        docker-compose down
                        docker-compose up -d --no-cache

                        # Verify that image tags were updated
                        echo "Verifying image tags in docker-compose.yaml and Kubernetes files:"
                        grep 'image:' docker-compose.yaml
                        grep 'image:' ./k8s/backend-deployment.yaml
                        grep 'image:' ./k8s/frontend-deployment.yaml

                        EOF
                        """
                    }
                }
            }
        }
    }
        //   Update docker-compose.yaml with new IMAGE_TAG
                     sh "sed -i 's#image: ${backendImageName}:.*#image: ${backendImageName}:${IMAGE_TAG}#' ./docker-compose.yaml"
                     sh "sed -i 's#image: ${frontendImageName}:.*#image: ${frontendImageName}:${IMAGE_TAG}#' ./docker-compose.yaml"


                    // Configure Git for committing the changes
                    sh 'git config --global user.email "shashidas95@gmail.com"'
                    sh 'git config --global user.name "shashidas95"'
                    sh 'git add ./k8s/backend-deployment.yaml ./k8s/frontend-deployment.yaml'
                    sh 'git add docker-compose.yaml'
                    sh "git commit -m 'Updated docker-compose file, backend and frontend deployment files to ${IMAGE_TAG}'"
                }


    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed'
        }
    }

