pipeline {
    agent any
    tools {
        jdk 'Java17'
    }
    environment {
        PROJECT_NAME = "mz2mo-server"

        DISCORD_WEBHOOK_URL = "https://discord.com/api/webhooks/1083501571083599873/4prm5Aoo_G84zQjvDhQ-i4lTWipLHArrTFVVx9UxXf7rk5QDCk2FWoXRHkh6uXk_jzCt"

        SPRING_PROFILE = "test"
    }
    stages {
        stage('Prepare Permission') {
            steps {
                sh 'chmod +x gradlew'
            }
        }

        stage('Gradle Test') {
            steps {
                sh  './gradlew clean test'
            }
        }
    }

    post {
        always {
            sh "docker image prune -a -f"
        }
        success {
            discordSend description: "${env.BUILD_NUMBER}번째 어플리케이션 테스트에 성공했어요!",
              footer: "젠킨스 배포 알림",
              link: env.BUILD_URL, result: currentBuild.currentResult,
              title: "${env.PROJECT_NAME}#${env.BUILD_NUMBER}",
              webhookURL: env.DISCORD_WEBHOOK_URL
        }
        failure {
            discordSend description: "${env.BUILD_NUMBER}번째 어플리케이션 테스트에 실패했어요!",
              footer: "젠킨스 배포 알림",
              link: env.BUILD_URL, result: currentBuild.currentResult,
              title: "${env.PROJECT_NAME}#${env.BUILD_NUMBER}",
              webhookURL: env.DISCORD_WEBHOOK_URL
        }
    }
}
