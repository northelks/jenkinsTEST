pipeline {
    agent any
    tools { nodejs 'node' }

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
        stage('Lint Dockerfile') {
            steps { sh 'docker run --rm -i hadolint/hadolint < Dockerfile' }
        }
        stage('Build') {
            steps { sh 'npm install' }
        }
        stage('Test') {
            steps { sh 'CI=true npm test' }
        }
        stage('Docker build') {
            steps { sh "docker build --platform=linux/amd64 -t ${IMAGE_NAME}:${IMAGE_TAG} ." }
        }
        stage('Scan vulnerabilities via Trivy') {
            steps {
                script {
                    def report = sh(
                        script: "trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress ${IMAGE_NAME}:${IMAGE_TAG}",
                        returnStdout: true
                    ).trim()
                    echo "Vulnerability Report:\n${report}"
                }
            }
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