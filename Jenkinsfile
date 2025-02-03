pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'ashudurge/python-jenkins-ci-ashu:unknown-0'
    }
    stages {
        stage('Declarative: Checkout SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/Ashwini2593/MultibranchPipelineUsingPython.git'
                checkout scm
            }
        }
        
        stage('Initialize') {
            steps {
                script {
                    // Ensure BRANCH_NAME is not empty
                    env.BRANCH_NAME = env.BRANCH_NAME ?: 'default'
                    env.DOCKER_IMAGE_TAG = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                    echo "✅ BRANCH_NAME set to: ${env.BRANCH_NAME}"
                    echo "✅ Docker image tag: ${env.DOCKER_IMAGE_TAG}"
                }
            }
        }

        stage('Setup Python Environment') {
            steps {
                script {
                    sh '''
                    python3 -m venv venv
                    source venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                    '''
                    echo "✅ Python environment setup complete."
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    sh '''
                    source venv/bin/activate
                    pytest --junitxml=results.xml
                    '''
                    echo "✅ Tests executed successfully."
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                    docker build -t ${DOCKER_IMAGE} .
                    '''
                    echo "✅ Docker image built successfully."
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                     // Log in to Docker Hub
                    withCredentials([usernamePassword(credentialsId: 'dockerHub', usernameVariable: 'DOCKER_HUB_USERNAME', passwordVariable: 'DOCKER_HUB_PASSWORD')]) {
                        sh 'echo ${DOCKER_HUB_PASSWORD} | docker login -u ${DOCKER_HUB_USERNAME} --password-stdin'
                    }
                    // Push the image
                    sh 'docker push ${DOCKER_IMAGE}:${DOCKER_IMAGE_TAG}'
                    echo "✅ Docker image pushed to DockerHub."
                }
            }
        }

        stage('Run Docker Container & Capture Output') {
            steps {
                script {
                    echo "✅ Running Docker container and capturing output..."
                    sh '''
                    docker run --rm ${DOCKER_IMAGE} python3 app.py
                    '''
                    echo "✅ Application output displayed above."
                }
            }
        }

        stage('Declarative: Post Actions') {
            steps {
                echo "✅ Build and deployment successful!"
            }
        }
    }
}

