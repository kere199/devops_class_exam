pipeline {
    agent any

    environment {
        TARGET_HOST  = "172.16.0.3"
        APP_DIR      = "/home/laborant/my-nodejs-app"
        SERVICE_NAME = "sample-node"
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
                sh 'node --test'
            }
        }

        stage('Deploy to Target VM') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'mykey',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh '''
                    ssh -i "$SSH_KEY" -o StrictHostKeyChecking=no $SSH_USER@${TARGET_HOST} << EOF
                      mkdir -p ${APP_DIR}
                      cd ${APP_DIR}
                      if [ ! -d .git ]; then
                        git clone https://github.com/kere199/devops_class_exam .
                      else
                        git pull
                      fi
                      npm install
                      sudo systemctl restart ${SERVICE_NAME} || true
                    EOF
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t sample-node-app:latest .'
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
