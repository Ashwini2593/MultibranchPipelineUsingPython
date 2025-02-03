pipeline {
    agent any

    environment {
        DOCKER_IMAGE_TAG = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
        IMAGE_NAME = 'ashudurge/python-jenkins-ci-ashu'
    }

    stages {
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
                    '''
                    echo "✅ Tests executed successfully."
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME}:${DOCKER_IMAGE_TAG} ."
                    echo "✅ Docker image built successfully."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "dockerHub", passwordVariable: "dockerHubPass", usernameVariable: "dockerHubUser")]) {
                    script {
                        sh '''
                            docker login -u ${dockerHubUser} -p ${dockerHubPass}
                            docker tag ${IMAGE_NAME}:${DOCKER_IMAGE_TAG} ${dockerHubUser}/${IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                            docker push ${dockerHubUser}/${IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                        '''
                        echo "✅ Docker image pushed to DockerHub."
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build and deployment successful!"
            build job: 'downstream-job', wait: false
        }
        failure {
            echo "❌ Build failed, please check logs!"
        }
    }
}
