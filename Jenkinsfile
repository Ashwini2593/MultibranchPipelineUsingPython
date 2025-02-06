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
                    echo "✅ Current environment variables:"
                    sh 'printenv'  // Debugging step to print all environment variables
                }
            }
        }

        stage('Checkout') {
            steps {
                script {
                    echo "✅ Checking out the code from the repository..."
                    git branch: 'main', url: 'https://github.com/Ashwini2593/MultibranchPipelineUsingPython.git'
                    echo "✅ Code checkout completed."
                    sh 'ls -al'  // Debugging step to list files in the workspace
                }
            }
        }

        stage('Setup Python Environment') {
            steps {
                script {
                    echo "✅ Setting up Python environment..."
                    sh '''
                        python3 -m venv venv
                        source venv/bin/activate
                        pip install --upgrade pip  
                        pip install -r requirements.txt
                    '''
                    echo "✅ Python environment setup complete."
                    sh 'pip freeze'  // Debugging step to verify installed Python packages
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    echo "✅ Running tests..."
                    sh '''
                        source venv/bin/activate
                        PYTHONPATH=$(pwd) pytest --junitxml=results.xml
                    '''
                    echo "✅ Tests executed successfully....."
                    echo "✅ Test results:"
                    sh 'cat results.xml'  // Debugging step to output test results
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "✅ Building Docker image..."
                    sh "docker build -t ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_IMAGE_TAG} ."
                    echo "✅ Docker image built successfully...."
                    sh "docker images"  // Debugging step to list Docker images
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    echo "✅ Logging into Docker Hub..."
                    withCredentials([usernamePassword(credentialsId: 'dockerHub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                    }
                    echo "✅ Pushing Docker image to Docker Hub..."
                    sh "docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_IMAGE_TAG}"
                    echo "✅ Docker image pushed to Docker Hub...."
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
            echo "✅ Build and deployment successful!"
            script {
                emailext(
                    subject: "✅ SUCCESS: Jenkins Job '${env.JOB_NAME}' [${env.BUILD_NUMBER}]",
                    body: """<h2>Build Successful!</h2>
                             <p>Job: ${env.JOB_NAME}</p>
                             <p>Build Number: ${env.BUILD_NUMBER}</p>
                             <p>Status: SUCCESS ✅</p>
                             <p><a href="${env.BUILD_URL}">Click here to view build details</a></p>""",
                    to: "${RECIPIENT_EMAIL}",
                    mimeType: 'text/html'
                )
            }
        }

        failure {
            echo "❌ Build failed, please check logs."
            script {
                emailext(
                    subject: "❌ FAILURE: Jenkins Job '${env.JOB_NAME}' [${env.BUILD_NUMBER}]",
                    body: """<h2>Build Failed!</h2>
                             <p>Job: ${env.JOB_NAME}</p>
                             <p>Build Number: ${env.BUILD_NUMBER}</p>
                             <p>Status: FAILURE ❌</p>
                             <p><a href="${env.BUILD_URL}">Click here to view build details</a></p>""",
                    to: "${RECIPIENT_EMAIL}",
                    mimeType: 'text/html'
                )
            }
        }
    }
}
