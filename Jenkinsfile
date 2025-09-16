pipeline {
    agent any
    
    tools {
        nodejs 'node'
    }

    environment {
        IMAGE_TAG = 'v1.0'
        BASE_URL = "http://localhost"
        APP_PORT = '' 
        IMAGE_NAME = ''
    }

    stages {
        stage('Initialize Build') {
            steps {
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

        stage('Checkout & Build Application') {
            steps {
                checkout scm
                sh "npm install"
            }
        }
        
        stage('Test Application') {
            steps {
                sh "npm test"
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image ${env.IMAGE_NAME}:${env.IMAGE_TAG}..."
                sh "docker build -t ${env.IMAGE_NAME}:${env.IMAGE_TAG} ."
            }
        }

        stage('Deploy Application') {
            steps {
                echo "Stopping and removing old container for ${env.IMAGE_NAME}..."
                sh "docker stop ${env.IMAGE_NAME} || true"
                sh "docker rm ${env.IMAGE_NAME} || true"

                echo "Deploying new container for ${env.IMAGE_NAME} on port ${env.APP_PORT}..."
                
                sh "docker run -d --name ${env.IMAGE_NAME} -p ${env.APP_PORT}:${env.APP_PORT} -e PORT=${env.APP_PORT} ${env.IMAGE_NAME}:${env.IMAGE_TAG}"

                echo "Application deployed at ${env.BASE_URL}:${env.APP_PORT}"
            }
        }
    }


    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
