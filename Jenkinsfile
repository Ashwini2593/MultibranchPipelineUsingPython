pipeline {
    agent any

    environment {
       // DOCKER_IMAGE_TAG = "unknown-0"  // Temporary default value
        DOCKER_REGISTRY = "docker.io"
        DOCKER_IMAGE = "ashudurge/python-jenkins-ci-ashu"
    }

    stages {
        stage('Initialize') {
            steps {
                script {
                    // Ensure BRANCH_NAME is not empty
                    env.BRANCH_NAME = env.BRANCH_NAME ?: 'default'
                    //env.DOCKER_IMAGE_TAG = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                    echo "✅ BRANCH_NAME set to: ${env.BRANCH_NAME}"
                    echo "✅ Docker image tag: ${env.DOCKER_IMAGE_TAG}"
                }
            }
        }

        stage('Checkout') {
            steps {
                script {
                    git branch: 'main', url: 'https://github.com/Ashwini2593/MultibranchPipelineUsingPython.git'
                    echo "✅ Code checkout completed................................"
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
                    echo "✅ Python environment setup complete............."
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    sh '''
                        source venv/bin/activate
                        PYTHONPATH=$(pwd) pytest --junitxml=results.xml
                    '''
                    echo "✅ Tests executed successfully..................."
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageTag = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                      sh "docker build -t ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${imageTag} ."
                   // sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_IMAGE_TAG} ."
                    echo "✅ Docker image built successfully..................."
                }
            }
        }
 
        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    // Log in to Docker Hub
                    withCredentials([usernamePassword(credentialsId: 'dockerHub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin'
                    }
                    // Push the image
                    sh 'docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${imageTag}'
                    echo "✅ Docker image pushed to DockerHub..............."
                }
            }
        }

        stage('Run Docker Container & Capture Output') {
            steps {
                script {
                    echo "✅ Running Docker container and capturing output..."
                    sh '''
                    docker run --rm ${DOCKER_IMAGE}:${DOCKER_IMAGE_TAG} python3 app.py
                    '''
                    echo "✅ Application output displayed above............"
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build and deployment successful.......!"
        }
        failure {
            echo "❌ Build failed, please check logs.......!"
        }
    }
}
