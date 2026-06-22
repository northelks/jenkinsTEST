pipeline {
    agent any
    tools { nodejs 'node' }

    environment {
        REGISTRY   = 'northelks'
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
            steps { sh "docker build --platform=linux/amd64 -t ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ." }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'epamlab-dockerhub',
                                usernameVariable: 'LAB_USER', passwordVariable: 'LAB_PASS')]) {
                    sh 'echo "$LAB_PASS" | docker login -u "$LAB_USER" --password-stdin'
                    sh "docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }
        stage('Scan vulnerabilities via Trivy') {
            steps {
                script {
                    def report = sh(
                        script: "trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}",
                        returnStdout: true
                    ).trim()
                    echo "Vulnerability Report:\n${report}"
                }
            }
        }
        stage('Deploy') {
            steps {
                sh """
                    docker ps -aq --filter "ancestor=${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}" | xargs -r docker rm -f
                    docker pull ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                    docker run -d -it --platform=linux/amd64 -p ${HOST_PORT}:${CONTAINER_PORT} ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
        stage('Deploy to DEV/MAIN') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'dev') {
                        build job: 'Deploy_to_dev', wait: false, parameters: [string(name: 'IMAGE_TAG', value: IMAGE_TAG)]
                    } else {
                        build job: 'Deploy_to_main', wait: false, parameters: [string(name: 'IMAGE_TAG', value: IMAGE_TAG)]
                    }
                }
            }
        }
    }
}