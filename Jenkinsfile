pipeline {
    agent any
    tools {
        nodejs 'node'
    }
    environment {
        IMAGE_TAG = 'v1.0'
        BASE_URL = "http://localhost"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
                script {
                    if (env.BRANCH_NAME == 'dev') {
                        env.APP_PORT = '3001'
                        env.IMAGE_NAME = 'nodedev'
                    } else if (env.BRANCH_NAME == 'main') {
                        env.APP_PORT = '3000'
                        env.IMAGE_NAME = 'nodemain'
                    }
                    echo "Branch: ${env.BRANCH_NAME}, App Port: ${env.APP_PORT}, Image Name: ${env.IMAGE_NAME}"
                }
            }
        }

        stage('Build & Test Application') {
            steps {
                script {
                    def port = '3000'
                    def image = 'nodemain'
                    if (env.BRANCH_NAME == 'dev') {
                        port = '3001'
                        image = 'nodedev'
                    } else if (env.BRANCH_NAME == 'main') {
                        port = '3000'
                        image = 'nodemain'
                    }
                    sh "echo Running npm install"
                    sh "npm install"
                    sh "npm test"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def port = '3000'
                    def image = 'nodemain'
                    if (env.BRANCH_NAME == 'dev') {
                        port = '3001'
                        image = 'nodedev'
                    } else if (env.BRANCH_NAME == 'main') {
                        port = '3000'
                        image = 'nodemain'
                    }

                    echo "Building Docker image ${image}:${env.IMAGE_TAG}..."
                    sh "docker build -t ${image}:${env.IMAGE_TAG} ."
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    def port = '3000'
                    def image = 'nodemain'
                    if (env.BRANCH_NAME == 'dev') {
                        port = '3001'
                        image = 'nodedev'
                    } else if (env.BRANCH_NAME == 'main') {
                        port = '3000'
                        image = 'nodemain'
                    }

                    echo "Stopping and removing old container for ${image}..."
                    sh "docker stop ${image} || true"
                    sh "docker rm ${image} || true"

                    echo "Deploying new container for ${image} on port ${port}..."
                    sh "docker run -d --name ${image} -p ${port}:${port} ${image}:${env.IMAGE_TAG}"

                    echo "Application deployed at ${env.BASE_URL}:${port}"
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
