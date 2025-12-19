pipeline {
    agent any

    environment {
        TARGET_HOST = "172.16.0.3"
        APP_DIR     = "/home/laborant/sample-node-app"
        GIT_REPO    = "https://github.com/kere199/devops_class_exam.git"
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
                    ssh -i "$SSH_KEY" -o StrictHostKeyChecking=no $SSH_USER@172.16.0.3 "
                        rm -rf /home/laborant/sample-node-app &&
                        git clone https://github.com/kere199/devops_class_exam.git /home/laborant/sample-node-app &&
                        cd /home/laborant/sample-node-app &&
                        npm install &&
                        nohup node index.js > app.log 2>&1 &
                    "
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            when {
                expression { sh(script: 'which docker', returnStatus: true) == 0 }
            }
            steps {
                sh 'docker build -t sample-node-app:latest .'
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                expression { sh(script: 'which kubectl', returnStatus: true) == 0 }
            }
            steps {
                sh '''
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }
}
