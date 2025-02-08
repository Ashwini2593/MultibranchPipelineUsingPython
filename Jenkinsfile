pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = "docker.io"
        DOCKER_IMAGE = "ashudurge/python-jenkins-ci-ashu"
        RECIPIENT_EMAIL = "adurge66@gmail.com"
    }

    stages {
        stage('Initialize') {
            steps {
                script {
                    env.BRANCH_NAME = env.BRANCH_NAME ?: 'default'
                    env.DOCKER_IMAGE_TAG = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                    echo "✅ BRANCH_NAME set to: ${env.BRANCH_NAME}"
                    echo "✅ Docker image tag: ${env.DOCKER_IMAGE_TAG}"
                }
            }
        }

        stage('Checkout') {
            steps {
                script {
                    git branch: 'main', url: 'https://github.com/Ashwini2593/MultibranchPipelineUsingPython.git'
                    echo "✅ Code checkout completed...."
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
                        PYTHONPATH=$(pwd) pytest --junitxml=results.xml
                    '''
                    echo "✅ Tests executed successfully........"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_IMAGE_TAG} ."
                    echo "✅ Docker image built successfully...."
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerHub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                    }
                    sh "docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_IMAGE_TAG}"
                    echo "✅ Docker image pushed to Docker Hub......"
                }
            }
        }

        stage('Run Docker Container & Capture Output') {
            steps {
                script {
                    echo "✅ Running Docker container and capturing output..."
                    sh "docker run --rm ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_IMAGE_TAG} python3 app.py"
                    echo "✅ Application output displayed above...."
                }
            }
        }
    }

    post {
        success {
            script {
                echo "Build succeeded. The Application is running successful."
            }
            mail to: 'adurge66@gmail.com', 
                 subject: 'Jenkins Job Succeeded', 
                 body: 'The Jenkins job has successfully completed......'
        }
        
        failure {
            mail to: 'adurge66@gmail.com', 
                 subject: 'Jenkins Job Failed', 
                 body: 'The Jenkins job has failed. Please check the logs.'
        }
    }
}
