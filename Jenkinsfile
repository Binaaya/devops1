pipeline {
    agent any
    
    environment {
        DOCKER_HOST = "npipe:////./pipe/docker_engine" // Windows Docker endpoint
        DOCKER_IMAGE = "Binaaya/my-web-app"
        DOCKER_TAG = "${env.BUILD_ID ?: 'latest'}"
        CONTAINER_NAME = "my-web-app-${env.BUILD_NUMBER}"
        GOOGLE_CHAT_WEBHOOK = credentials('google-chat-webhook') // Secure webhook
        DEPLOYMENT_URL = "http://localhost:8085"
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-credentials')
        HOST_PORT = "8085"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/Binaaya/devops1.git',
                    credentialsId: ''
            }
        }
        
        stage('Deploy') {
            steps {
                echo "deployed"
        }
    }

    post {
        always {
            echo "post "
        }
    
        success {
            node('built-in') {
                script {
                    def message = """
                    🚀 *Deployment Successful* 
                    *Build*: #${env.BUILD_NUMBER}
                    *Image*: ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}
                    *Container*: ${env.CONTAINER_NAME}
                    """
                    sendGoogleChatNotification(message)
                }
            }
        }
        failure {
            node('built-in') {
                script {
                    def logs = bat(
                        script: """
                            docker container inspect %CONTAINER_NAME% > nul 2>&1
                            if %ERRORLEVEL% == 0 (
                                docker logs --tail 50 %CONTAINER_NAME% 2>&1 || exit 0
                            ) else (
                                echo Container %CONTAINER_NAME% does not exist
                            )
                        """,
                        returnStdout: true
                    ).trim()
                    
                    def message = """
                    🔴 *Deployment Failed* 
                    *Build*: #${env.BUILD_NUMBER}
                    *Error*: ${currentBuild.currentResult}
                    *Logs*: ${logs}
                    """
                    sendGoogleChatNotification(message)
                }
            }
        }
    }
}

def sendGoogleChatNotification(String message) {
    node('built-in') {
        // Escape message for JSON and batch
        def escapedMessage = message.replace('"', '""').replace('\n', '^n').replace('%', '%%').replace('&', '^&')
        def payload = """{"text":"${escapedMessage}"}"""
        
        bat """
            curl -X POST ^
            -H "Content-Type: application/json" ^
            -d "${payload}" ^
            "%GOOGLE_CHAT_WEBHOOK%" || echo Notification failed
        """
    }
}
