pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "naveen93mm/html-app"
        DOCKER_TAG = "${BUILD_NUMBER}"
        DEPLOY_SERVER = "13.127.225.75"
    }

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main',
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
                    ssh -o StrictHostKeyChecking=no ec2-user@${DEPLOY_SERVER} '
                        docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker stop htmlapp || true
                        docker rm htmlapp || true
                        docker run -d -p 9091:80 --name htmlapp ${DOCKER_IMAGE}:${DOCKER_TAG}
                    '
                    """
                }
            }
        }
    }
}
