pipeline {
    agent any
    tools {
        nodejs 'node'
    }
    environment {
        APP_PORT = '3000'
        IMAGE_NAME = 'nodemain'
        IMAGE_TAG = 'v1.0'
        BASE_URL = "http://localhost"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                script {
                    def appPort = '3000'
                    def imageName = 'nodemain'

                    if (env.BRANCH_NAME == 'dev') {
                        appPort = '3001'
                        imageName = 'nodedev'
                    } else if (env.BRANCH_NAME == 'main') {
                        appPort = '3000'
                        imageName = 'nodemain'
                    }
                    env.APP_PORT = appPort
                    env.IMAGE_NAME = imageName

                    echo "Branch: ${env.BRANCH_NAME}, App Port: ${env.APP_PORT}, Image Name: ${env.IMAGE_NAME}"
                }

            }
        }

        stage('Build & Test Application') {
            steps {
                sh 'npm install'
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image ${IMAGE_NAME}:${IMAGE_TAG}..."
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Deploy Application') {
            stage('Build Docker Image') {
                steps {
                    script {
                        withEnv(["APP_PORT=${env.APP_PORT}", "IMAGE_NAME=${env.IMAGE_NAME}"]) {
                            sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                            sh "docker stop ${IMAGE_NAME} || true"
                            sh "docker rm ${IMAGE_NAME} || true"
                            sh "docker run -d --name ${IMAGE_NAME} -p ${APP_PORT}:${APP_PORT} ${IMAGE_NAME}:${IMAGE_TAG}"
                        }
                    }
                }
            }

        }
    }

    post {
        always {
            echo 'Pipeline finished.'
            cleanWs()
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
