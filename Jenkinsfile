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
                echo "Code clone ho gaya hai"
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    // Create and activate a virtual environment
                    sh 'python3 -m venv venv'
                    sh 'source venv/bin/activate'  // For Linux-based systems
                    
                    // Ensure pip is up to date
                    sh 'python3 -m pip install --upgrade pip'

                    // Install dependencies inside the virtual environment
                    sh 'python3 -m pip install -r requirements.txt'
                    sh 'python3 setup.py install'
                }
            }
        }

        stage('Test') {
            steps {
                sh 'pytest'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${DOCKER_IMAGE_TAG} ."
                echo "Docker build bhi ho chuka hai"
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "dockerHub", passwordVariable: "dockerHubPass", usernameVariable: "dockerHubUser")]) {
                    sh "docker login -u ${dockerHubUser} -p ${dockerHubPass}"
                    sh "docker tag ${IMAGE_NAME}:${DOCKER_IMAGE_TAG} ${dockerHubUser}/${IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                    sh "docker push ${dockerHubUser}/${IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                    echo "DockerHub pe push ho gaya hai"
                }
            }
        }
    }

    post {
        success {
            build job: 'downstream-job', wait: false
            mail to: 'adurge66@gmail.com',
                 subject: "Build Success: ${env.JOB_NAME} ${env.BUILD_NUMBER}",
                 body: "The build was successful! Visit the job at ${env.BUILD_URL}"
        }
        failure {
            mail to: 'adurge66@gmail.com',
                 subject: "Build Failure: ${env.JOB_NAME} ${env.BUILD_NUMBER}",
                 body: "The build failed. Check the job at ${env.BUILD_URL}"
        }
    }
}
