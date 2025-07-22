pipeline {
    agent any

    environment {
        DOCKER_HOST = "npipe:////./pipe/docker_engine" // Windows Docker endpoint
        DOCKER_IMAGE = "Binaaya/my-web-app"
        DEPLOYMENT_URL = "http://localhost:8085"
        HOST_PORT = "8085"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/Binaaya/devops1.git'
                // Add credentialsId if needed
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Just a dummy deploy step for now
                    env.DOCKER_TAG = env.BUILD_ID ?: 'latest'
                    env.CONTAINER_NAME = "my-web-app-${env.BUILD_NUMBER}"
                    echo "Deploying container ${env.CONTAINER_NAME}"
                }
            }
        }
    }

    post {
        always {
            echo "post step executed"
        }

        success {
            script {
                withCredentials([string(credentialsId: 'google-chat-webhook', variable: 'GOOGLE_CHAT_WEBHOOK')]) {
                    def message = """
                    🚀 *Deployment Successful* 
                    *Build*: #${env.BUILD_NUMBER}
                    *Image*: ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}
                    *Container*: ${env.CONTAINER_NAME}
                    """
                    sendGoogleChatNotification(message, GOOGLE_CHAT_WEBHOOK)
                }
            }
        }

        failure {
            script {
                def logs = bat(
                    script: """
                        docker container inspect ${env.CONTAINER_NAME} > nul 2>&1
                        if %ERRORLEVEL% == 0 (
                            docker logs --tail 50 ${env.CONTAINER_NAME} 2>&1 || exit 0
                        ) else (
                            echo Container ${env.CONTAINER_NAME} does not exist
                        )
                    """,
                    returnStdout: true
                ).trim()

                withCredentials([string(credentialsId: 'google-chat-webhook', variable: 'GOOGLE_CHAT_WEBHOOK')]) {
                    def message = """
                    🔴 *Deployment Failed* 
                    *Build*: #${env.BUILD_NUMBER}
                    *Error*: ${currentBuild.currentResult}
                    *Logs*: ${logs}
                    """
                    sendGoogleChatNotification(message, GOOGLE_CHAT_WEBHOOK)
                }
            }
        }
    }
}

def sendGoogleChatNotification(String message, String webhook) {
    def escapedMessage = message.replace('"', '""').replace('\n', '^n').replace('%', '%%').replace('&', '^&')
    def payload = """{"text":"${escapedMessage}"}"""

    bat """
        curl -X POST ^
        -H "Content-Type: application/json" ^
        -d "${payload}" ^
        "${webhook}" || echo Notification failed
    """
}
