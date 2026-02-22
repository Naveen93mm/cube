pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "naveen93mm/html-app"
        DOCKER_TAG = "${BUILD_NUMBER}"
        DEPLOY_SERVER = "65.2.180.201"
    }

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'update',
                    url: 'https://github.com/Naveen93mm/cube.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                """
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                sh """
                docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                """
            }
        }

       stage('Deploy to VM') {
            steps {
                sshagent(['vm-ssh-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ec2-user@65.2.180.201 '
                        docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker stop htmlapps || true
                        docker rm htmlapps || true
                        docker run -d -p 9092:80 --name htmlapps ${DOCKER_IMAGE}:${DOCKER_TAG}
                    '
                    """
                }
            }
        }
    }
}
