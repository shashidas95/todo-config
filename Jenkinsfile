pipeline {
    agent any

    parameters {
        string(name: 'IMAGE_TAG', description: 'Enter the image tag')
    }
    
    environment {
        GIT_REPO = "https://github.com/shashidas95/todo-config"
       // GOOGLE_CHAT_WEBHOOK = "YOUR_GOOGLE_CHAT_WEBHOOK_URL"  // Replace with your actual webhook URL
    }

    stages {
        stage('CLEANUP WORKSPACE') {
            steps {
                cleanWs()
            }
        }

        stage("CHECKOUT GIT REPO") {
            steps {
                git branch: 'main', url: "${GIT_REPO}"
            }
        }

        stage("UPDATE K8S DEPLOYMENT FILES") {
            steps {
                script {
                    def backendImageName = "${DOCKERHUB_USERNAME}/todo-be"
                    def frontendImageName = "${DOCKERHUB_USERNAME}/todo-fe"
                    def imageTag = params.IMAGE_TAG

                    // Update backend and frontend deployment files
                    sh "sed -i 's#image: ${backendImageName}:.*#image: ${backendImageName}:${imageTag}#' ./k8s/backend-deployment.yaml"
                    sh "sed -i 's#image: ${frontendImageName}:.*#image: ${frontendImageName}:${imageTag}#' ./k8s/frontend-deployment.yaml"

                    // Update docker-compose.yml with new IMAGE_TAG
                    sh "sed -i 's#image: ${backendImageName}:.*#image: ${backendImageName}:${imageTag}#' ./docker-compose.yml"
                    sh "sed -i 's#image: ${frontendImageName}:.*#image: ${frontendImageName}:${imageTag}#' ./docker-compose.yml"

                    // Configure Git and commit changes
                    sh 'git config --global user.email "shashidas95@gmail.com"'
                    sh 'git config --global user.name "shashidas95"'
                    sh 'git add ./k8s/backend-deployment.yaml ./k8s/frontend-deployment.yaml ./docker-compose.yml'
                    sh "git commit -m 'Updated deployment files and docker-compose with IMAGE_TAG: ${imageTag}'"
                    
                    withCredentials([usernamePassword(credentialsId: 'githubuser', passwordVariable: 'pass', usernameVariable: 'uname')]) {
                        sh 'git push https://$uname:$pass@github.com/shashidas95/todo-config.git main'
                    }

                    // Notify Google Chat
                    //notifyGoogleChat(imageTag)
                }
            }
        }
    }

    post {
        success {
            echo 'K8s and Docker Compose Update Pipeline completed successfully'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}

//def notifyGoogleChat(imageTag) {
   // def webhookUrl = env.GOOGLE_CHAT_WEBHOOK
   // def message = [
   //     "text": "Image tag updated to ${imageTag}"
   // ]
   // httpRequest httpMode: 'POST', contentType: 'APPLICATION_JSON', requestBody: groovy.json.JsonOutput.toJson(message), url: webhookUrl
//}
