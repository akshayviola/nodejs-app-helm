pipeline {
    agent any
    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        GIT_CREDENTIALS_ID = '36ff0bcd-3a76-47d6-a5a7-7315f966b7ba' // Git credentials ID
        IMAGE_NAME = "akshay2001a/nodejs-app"
        GIT_REPO = 'https://github.com/akshayviola/nodejs-app-helm.git'
        GIT_BRANCH = 'main'
        HELM_CHART_PATH = '.' // Set this to the path of your Helm chart in the Git repo
    }
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    def imageTag = "${env.BUILD_NUMBER}"
                    docker.build("${env.IMAGE_NAME}:${imageTag}", "app")
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
        stage('Update Helm Chart') {
            steps {
                script {
                    def imageTag = "${env.BUILD_NUMBER}"
                    
                    // Ensure the correct path to values.yaml
                    sh """
                    sed -i 's|image: .*|image: ${IMAGE_NAME}:${imageTag}|' ${HELM_CHART_PATH}/values.yaml
                    """
                }
            }
        }
        stage('Commit Changes') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: env.GIT_CREDENTIALS_ID, usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                        sh """
                        git config user.name "akshayviola"
                        git config user.email "akshaysunil201@gmail.com"
                        git checkout ${GIT_BRANCH} || git checkout -b ${GIT_BRANCH}  // Checkout the branch
                        git add ${HELM_CHART_PATH}/values.yaml
                        git commit -m "Update image tag to ${BUILD_NUMBER}" || echo "No changes to commit"
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@${GIT_REPO} ${GIT_BRANCH}
                        """
                    }
                }
            }
        }
        stage('Apply Helm Release') {
            steps {
                script {
                    // Apply the HelmRelease configuration using FluxCD
                    sh 'flux reconcile source git helm-repo'
                    sh 'flux reconcile helmrelease nodejs-app'
                }
            }
        }
    }
}
