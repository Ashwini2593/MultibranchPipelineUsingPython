pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = "docker.io"
        DOCKER_IMAGE = "ashudurge/python-jenkins-ci-ashu"
        RECIPIENT_EMAIL = "adurge66@gmail.com"  // Your Gmail address
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
                    echo "✅ Code checkout completed."
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
                        deactivate
                    '''
                    echo "✅ Tests executed successfully......."
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_IMAGE_TAG} ."
                    echo "✅ Docker image built successfully."
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
                    echo "✅ Docker image pushed to Docker Hub."
                }
            }
        }

        stage('Run Docker Container & Capture Output') {
            steps {
                script {
                    echo "✅ Running Docker container and capturing output..."
                    sh "docker run --rm ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_IMAGE_TAG} python3 app.py"
                    echo "✅ Application output displayed above."
                }
            }
        }
    }

    post {
        always {
            script {
                echo "📨 Sending build notification email ........."
                try {
                    mail bcc: '',
                         cc: '',
                         from: 'adurge66@gmail.com',
                         replyTo: 'adurge66@gmail.com',
                         to: "${RECIPIENT_EMAIL}",
                         subject: "[Jenkins] Job '${env.JOB_NAME}' #${env.BUILD_NUMBER}",
                         body: """Jenkins build results:

🔹 **Job Name**: ${env.JOB_NAME}
🔹 **Build Number**: ${env.BUILD_NUMBER}
🔹 **Build Status**: ${currentBuild.currentResult}
🔹 **Triggered By**: ${currentBuild.getBuildCauses().toString()}

Check the logs for details: ${env.BUILD_URL}"""
                    echo "✅ Email notification sent successfully."
                } catch (Exception e) {
                    echo "❌ Failed to send email: ${e.getMessage()}"
                }
            }
        }
    }
}
