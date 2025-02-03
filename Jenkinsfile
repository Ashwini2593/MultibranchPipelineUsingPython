pipeline {
    agent any
    environment {
        DOCKER_IMAGE_TAG = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
        IMAGE_NAME = 'ashudurge/python-jenkins-ci-ashu'
    }
    stages {
        stage('Checkout') {
            steps {
                // Clone the repository from GitHub
                 git url: "https://github.com/Ashwini2593/node-todo-cicd.git", branch: 'main'
                echo "Code clone ho gaya hai"
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    // Install dependencies and build the Python project
                    sh 'pip install -r requirements.txt'
                    sh 'python setup.py install'  // If you have setup.py for your project
                }
            }
        }

        stage('Test') {
            steps {
                // Run unit tests using pytest
                sh 'pytest'
            }
        }

        stage('Build Docker Image') {
            steps {
                // Build Docker image with the tag
                sh "docker build -t ${IMAGE_NAME}:${DOCKER_IMAGE_TAG} ."
                echo "docker build bhi ho chuka hai"
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "dockerHub", passwordVariable: "dockerHubPass", usernameVariable: "dockerHubUser")]) {
                    // Docker login
                    sh "docker login -u ${dockerHubUser} -p ${dockerHubPass}"
                    
                    // Tag and Push the Docker image to Docker Hub
                    sh "docker tag ${IMAGE_NAME}:${DOCKER_IMAGE_TAG} ${dockerHubUser}/${IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                    sh "docker push ${dockerHubUser}/${IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                    echo "DockerHub pe push ho gaya hai"
                }
            }
        }
    }

    post {
        success {
            // Trigger downstream job after successful build
            build job: 'downstream-job', wait: false
            
            // Send success email notification
            mail to: 'adurge66@gmail.com',
                 subject: "Build Success: ${env.JOB_NAME} ${env.BUILD_NUMBER}",
                 body: "The build was successful! Visit the job at ${env.BUILD_URL}"
        }
        failure {
            // Send failure email notification
            mail to: 'adurge66@gmail.com',
                 subject: "Build Failure: ${env.JOB_NAME} ${env.BUILD_NUMBER}",
                 body: "The build failed. Check the job at ${env.BUILD_URL}"
        }
    }
}
