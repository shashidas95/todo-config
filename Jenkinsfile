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
            sshagent(credentials: ['docker_server_credentials']) {
                sh """
                ssh -o StrictHostKeyChecking=no ${DOCKER_SERVER} 'bash -s' << EOF
                
                # Export necessary environment variables to be used in the remote session
                export DOCKERHUB_USERNAME=${DOCKERHUB_USERNAME}
                export IMAGE_BE=${IMAGE_BE}
                export IMAGE_FE=${IMAGE_FE}
                export IMAGE_TAG=${IMAGE_TAG}
                export CONFIG_PROJECT_NAME=${CONFIG_PROJECT_NAME}
                
                # Navigate to project directory
                cd ${CONFIG_PROJECT_NAME}

                # Pull latest changes
                git pull origin main

                # Update backend and frontend image tags in Kubernetes files and docker-compose.yaml
                sed -i "s#image: \${DOCKERHUB_USERNAME}/\${IMAGE_BE}:.*#image: \${DOCKERHUB_USERNAME}/\${IMAGE_BE}:\${IMAGE_TAG}#" ./k8s/backend-deployment.yaml
                sed -i "s#image: \${DOCKERHUB_USERNAME}/\${IMAGE_FE}:.*#image: \${DOCKERHUB_USERNAME}/\${IMAGE_FE}:\${IMAGE_TAG}#" ./k8s/frontend-deployment.yaml
                sed -i "s#image: \${DOCKERHUB_USERNAME}/\${IMAGE_BE}:.*#image: \${DOCKERHUB_USERNAME}/\${IMAGE_BE}:\${IMAGE_TAG}#" docker-compose.yaml
                sed -i "s#image: \${DOCKERHUB_USERNAME}/\${IMAGE_FE}:.*#image: \${DOCKERHUB_USERNAME}/\${IMAGE_FE}:\${IMAGE_TAG}#" docker-compose.yaml

                # Commit and push the changes if any
                git add ./k8s/backend-deployment.yaml ./k8s/frontend-deployment.yaml docker-compose.yaml
                git commit -m "Updated deployment files with new image tag \${IMAGE_TAG}" || echo "No changes to commit"
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

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}
