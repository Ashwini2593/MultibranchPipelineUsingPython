pipeline {
    agent any

    environment {
        BRANCH_NAME = env.BRANCH_NAME ?: 'default'  // Fixing null branch name issue
        DOCKER_IMAGE_TAG = "${BRANCH_NAME}-${env.BUILD_NUMBER}"
        IMAGE_NAME = "ashudurge/python-jenkins-ci-ashu"
    }

    stages {
        stage('Checkout the code ') {
            steps {
                script {
                    git branch: 'main', url: 'https://github.com/Ashwini2593/MultibranchPipelineUsingPython.git'
                    echo "✅ Code checkout completed........."
                }
            }
        }

        stage('Setup Python Environment is ready') {
            steps {
                script {
                    sh '''
                        python3 -m venv venv
                        source venv/bin/activate
                        pip install --upgrade pip  
                        pip install -r requirements.txt
                    '''
                    echo "✅ Python environment setup complete.........."
                }
            }
        }

        stage('Run Tests is completed') {
            steps {
                script {
                    sh '''
                        source venv/bin/activate
                        PYTHONPATH=$(pwd) pytest --junitxml=results.xml
                    '''
                    echo "✅ Tests executed successfully........."
                }
            }
        }

        stage('Build Docker Image is completed') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME}:${DOCKER_IMAGE_TAG} ."
                    echo "✅ Docker image built successfully.........."
                }
            }
        }

        stage('Push Docker Image to mydockerhub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "dockerHub", passwordVariable: "dockerHubPass", usernameVariable: "dockerHubUser")]) {
                    script {
                        sh '''
                            docker login -u ${dockerHubUser} -p ${dockerHubPass}
                            docker tag ${IMAGE_NAME}:${DOCKER_IMAGE_TAG} ${dockerHubUser}/${IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                            docker push ${dockerHubUser}/${IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                        '''
                        echo "✅ Docker image pushed to DockerHub successfully......"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build and deployment successfully!"
            build job: 'downstream-job', wait: false
        }
        failure {
            echo "❌ Build failed, please check logs!"
        }
    }
}
