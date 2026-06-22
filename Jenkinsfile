pipeline {
    agent any

    environment {
        IMAGE_NAME     = "${env.BRANCH_NAME == 'dev' ? 'nodedev' : 'nodemain'}"
        IMAGE_TAG      = "v1.0"
        HOST_PORT      = "${env.BRANCH_NAME == 'dev' ? '3001' : '3000'}"
        CONTAINER_PORT = "3000"
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Build') {
            agent { docker { image 'node:7.8.0'; args '--platform=linux/amd64'; reuseNode true } }
            steps { sh 'npm install' }
        }
        stage('Test') {
            agent { docker { image 'node:7.8.0'; args '--platform=linux/amd64'; reuseNode true } }
            environment { CI = 'true' }
            steps { sh 'npm test' }
        }
        stage('Docker build') {
            steps { sh "docker build --platform=linux/amd64 -t ${IMAGE_NAME}:${IMAGE_TAG} ." }
        }
        stage('Deploy') {
            steps {
                sh """
                    docker ps -aq --filter "ancestor=${IMAGE_NAME}:${IMAGE_TAG}" | xargs -r docker rm -f
                    docker run -d -it --platform=linux/amd64 --expose ${HOST_PORT} -p ${HOST_PORT}:${CONTAINER_PORT} ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
    }
}