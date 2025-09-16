@Library('shared-library') _

pipeline {
    agent any
    tools {
        nodejs 'node'
    }
    environment {
        IMAGE_TAG = 'v1.0'
        BASE_URL = "http://localhost"
        DOCKERHUB_CREDENTIALS = credentials('tamarabr-dockerhub')
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

        stage('Build & test with npm') {
            steps {
                script {
                    buildAndTest()
                }
            }
        }
        
        stage('Lint Dockerfile with Hadolint') {
            steps {
                runHadolint()
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

        


        stage('Scan for Vulnerabilities') {
            steps {
                script {
                    def fullImageName = "${DOCKERHUB_USERNAME}/${env.IMAGE_NAME}:${env.IMAGE_TAG}"
                    sh "docker tag ${env.IMAGE_NAME}:${env.IMAGE_TAG} ${fullImageName}"
                
                    scanWithTrivy(fullImageName: fullImageName)
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    dockerLogin(credentialsId: DOCKERHUB_CREDENTIALS_ID)
                    dockerTagAndPush(
                        imageName: env.IMAGE_NAME,
                        imageTag: env.IMAGE_TAG,
                        dockerhubUsername: DOCKERHUB_USERNAME
                    )
                }
            }
        }

        

    post {
        success {
            script {
                echo "CI build successful. Triggering CD process..."
                if (env.BRANCH_NAME == 'main') {
                    echo "Triggering Deploy_to_main with version ${IMAGE_TAG}"
                    build job: 'Deploy_to_main', parameters: [[$class: 'StringParameterValue', name: 'IMAGE_VERSION', value: env.IMAGE_TAG]]
                } else if (env.BRANCH_NAME == 'dev') {
                    echo "Triggering Deploy_to_dev with version ${IMAGE_TAG}"
                    build job: 'Deploy_to_dev', parameters: [[$class: 'StringParameterValue', name: 'IMAGE_VERSION', value: env.IMAGE_TAG]]
                }
            }
        }
        always {
            dockerLogout()
        }
    }
    
}
