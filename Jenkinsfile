pipeline {
    agent any
    environment {
        DOCKER_IMAGE_TAG = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
        IMAGE_NAME = 'ashudurge/python-jenkins-ci-ashu'
    }
    stages {
        stage('Checkout') {
            steps {
                git url: "https://github.com/Ashwini2593/MultibranchPipelineUsingPython.git", branch: 'main'
                echo "Code clone ho gaya hai............."
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    sh '''
                        python3 -m venv venv
                        source venv/bin/activate
                        pip install --upgrade pip  
                        pip install -r requirements.txt
                    '''
                }
            }
        }

        stage('Test') {
            steps {
                sh '''
                    source venv/bin/activate
                    PYTHONPATH=$(pwd) pytest
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${DOCKER_IMAGE_TAG} ."
                echo "Docker build bhi ho chuka hai..........."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "dockerHub", passwordVariable: "dockerHubPass", usernameVariable: "dockerHubUser")]) {
                    sh '''
                        docker login -u ${dockerHubUser} -p ${dockerHubPass}
                        docker tag ${IMAGE_NAME}:${DOCKER_IMAGE_TAG} ${dockerHubUser}/${IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                        docker push ${dockerHubUser}/${IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                    '''
                    echo "DockerHub pe push ho gaya hai"
                }
            }
        }
    }

    post {
        success {
            build job: 'downstream-job', wait: false
            echo "Build successful, skipping email for now"
        }
        failure {
            echo "Build failed, check logs"
        }
    }
}
