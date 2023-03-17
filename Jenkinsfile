pipeline {
    agent any
    tools {
        jdk 'Java17'
    }
    environment {
        PROJECT_NAME = "mz2mo-server"

        DISCORD_WEBHOOK_URL = "https://discord.com/api/webhooks/1086219891465539634/R2u8otPTZ7aFKBTSy-X3jZnGWkdifP-ltjI-0hyw9BIQDgES-aHLYI9MnmXjDRFnF3wJ"

        DOCKER_CREDENTIAL_ID = "docker"
        
        DEV_DOCKER_IMAGE_NAME = "mz2mo/mz2mo-server-dev"
        DEV_DOCKER_CONTAINER_NAME = "mz2mo-server-dev"
        DEV_DOCKER_ENV_FILE_CREDENTIAL_ID = 'docker-env-file-dev'

        PROD_DOCKER_IMAGE_NAME = "mz2mo/mz2mo-server-prod"
        PROD_DOCKER_CONTAINER_NAME = "mz2mo-server-prod"
        PROD_DOCKER_ENV_FILE_CREDENTIAL_ID = 'docker-env-file-prod'

        SPRING_PROFILE = "dev"
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

        stage('Gradle Build') {
            steps {
                sh './gradlew clean build -x test'
            }
        }

        stage('Develop Docker Build') {
            when {
                { it -> branch 'develop' }
            }
            steps {
                script {
                    app = docker.build(DEV_DOCKER_IMAGE_NAME)
                    docker.withRegistry('https://registry.hub.docker.com', DEV_DOCKER_CREDENTIAL_ID) {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }

        stage('Develop Docker Run') {
            when {
                expression { return env.BRANCH_NAME ==~ /develop/ }
            }
            steps {
                sh "docker ps -q --filter \"name=${DEV_DOCKER_CONTAINER_NAME}\" || grep -q . && docker stop ${DEV_DOCKER_CONTAINER_NAME} && docker rm ${DEV_DOCKER_CONTAINER_NAME} || true"
                sh "docker run -e SPRING_PROFILE=${env.SPRING_PROFILE} --env-file=${DEV_DOCKER_ENV_FILE_CREDENTIAL_ID} -p 8081:8080 --name=${DEV_DOCKER_CONTAINER_NAME}  -d ${DEV_DOCKER_IMAGE_NAME}"
            }
        }

        stage('Production Docker Build') {
            when {
                expression { return env.BRANCH_NAME ==~ /develop/ }
            }
            steps {
                script {
                    app = docker.build(DEV_DOCKER_IMAGE_NAME)
                    docker.withRegistry('https://registry.hub.docker.com', DEV_DOCKER_CREDENTIAL_ID) {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }

        stage('Production Docker Run') {
            when {
                expression { return env.BRANCH_NAME ==~ /release\/.*/ }
            }
            steps {
                sh "docker ps -q --filter \"name=${DEV_DOCKER_CONTAINER_NAME}\" || grep -q . && docker stop ${DEV_DOCKER_CONTAINER_NAME} && docker rm ${DEV_DOCKER_CONTAINER_NAME} || true"
                sh "docker run -e SPRING_PROFILE=${env.SPRING_PROFILE} --env-file=${DEV_DOCKER_ENV_FILE_CREDENTIAL_ID} -p 8081:8080 --name=${DEV_DOCKER_CONTAINER_NAME}  -d ${DEV_DOCKER_IMAGE_NAME}"
            }
        }
    }

    post {
        always {
            sh "docker image prune -a -f"
        }
        success {
            discordSend description: "${env.BUILD_NUMBER}번째 어플리케이션 배포에 성공했어요!",
              footer: "젠킨스 배포 알림",
              link: env.BUILD_URL, result: currentBuild.currentResult,
              title: "${env.PROJECT_NAME}#${env.BUILD_NUMBER}",
              webhookURL: env.DISCORD_WEBHOOK_URL
        }
        failure {
            discordSend description: "${env.BUILD_NUMBER}번째 어플리케이션 배포에 실패했어요!",
              footer: "젠킨스 배포 알림",
              link: env.BUILD_URL, result: currentBuild.currentResult,
              title: "${env.PROJECT_NAME}#${env.BUILD_NUMBER}",
              webhookURL: env.DISCORD_WEBHOOK_URL
        }
    }
}
