pipeline {
    agent any
    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        IMAGE_NAME = "akshay2001a/nodejs-app"
    }
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    def imageTag = "${env.BUILD_NUMBER}"
                    // Specify the Dockerfile location and build context
                    docker.build("${env.IMAGE_NAME}:${imageTag}", '-f app/Dockerfile .')
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', env.DOCKER_CREDENTIALS_ID) {
                        def imageTag = "${env.BUILD_NUMBER}"
                        docker.image("${env.IMAGE_NAME}:${imageTag}").push()
                    }
                }
            }
        }
    }
}
