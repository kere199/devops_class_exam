pipeline {
    agent any

    environment {
        TARGET_HOST  = "172.16.0.3"
        APP_DIR      = "/home/laborant/my-nodejs-app"
        SERVICE_NAME = "sample-node"
        DOCKER_IMAGE = "sample-node-app:latest"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                node -v
                npm install
                '''
            }
        }

        stage('Unit Tests') {
            steps {
                sh '''
                node --test
                '''
            }
        }

        stage('Deploy to Target VM') {
            steps {
                sshagent(credentials: ['mykey']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no laborant@${TARGET_HOST} << EOF
                      cd ${APP_DIR}
                      git pull
                      npm install
                      sudo systemctl restart ${SERVICE_NAME}
                    EOF
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t ${DOCKER_IMAGE} .
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }
}
