pipeline {
    agent any

    environment {
        DOCKER_IMAGE_TAG = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
        IMAGE_NAME = 'ashudurge/python-jenkins-ci-ashu'
        DOCKER_HUB_REPO = 'ashudurge'
    }

    stages {
        stage('Checkout') {
            steps {
                cleanWs() // Clean workspace before each build
                git url: "https://github.com/Ashwini2593/MultibranchPipelineUsingPython.git", branch: 'main'
                echo "Code has been cloned successfully"
            }
        }

        stage('Build') {
            steps {
                script {
                    sh '''
                        python3 -m venv venv
                        source venv/bin/activate || . venv/bin/activate
                        pip install --upgrade pip
                        pip install -r requirements.txt
                    '''
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    sh '''
                        source venv/bin/activate || . venv/bin/activate
                        set -e  # Exit if tests fail
                        pytest
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME}:${DOCKER_IMAGE_TAG} ."
                    sh "docker tag ${IMAGE_NAME}:${DOCKER_IMAGE_TAG} ${IMAGE_NAME}:latest"
                    echo "Docker image built successfully"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "dockerHub", passwordVariable: "DOCKER_HUB_PASS", usernameVariable: "DOCKER_HUB_USER")]) {
                    script {
                        sh "echo ${DOCKER_HUB_PASS} | docker login -u ${DOCKER_HUB_USER} --password-stdin"
                        sh "docker tag ${IMAGE_NAME}:${DOCKER_IMAGE_TAG} ${DOCKER_HUB_USER}/${IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                        sh "docker tag ${IMAGE_NAME}:${DOCKER_IMAGE_TAG} ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                        sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                        sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                        echo "Docker images pushed to Docker Hub"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Triggering downstream job..."
            build job: 'downstream-job', wait: false
            mail to: 'adurge66@gmail.com',
                 subject: "✅ Build Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "The build was successful! Visit the job at ${env.BUILD_URL}"
        }
        failure {
            mail to: 'adurge66@gmail.com',
                 subject: "❌ Build Failure: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "The build failed. Check the logs at ${env.BUILD_URL}"
        }
    }
}
