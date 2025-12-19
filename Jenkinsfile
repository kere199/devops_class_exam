pipeline {
    agent any

    environment {
        GIT_REPO = "https://github.com/kere199/devops_class_exam.git"

        TARGET_HOST = "172.16.0.3"
        TARGET_DIR  = "/home/laborant/sample-node-app"

        DOCKER_HOST = "docker"
        DOCKER_APP  = "/home/laborant/sample-node-app"

        KUBE_API = "https://kubernetes:6443"
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
                    ssh -i "$SSH_KEY" -o StrictHostKeyChecking=no ${SSH_USER}@${TARGET_HOST} "
                        rm -rf ${TARGET_DIR} &&
                        git clone ${GIT_REPO} ${TARGET_DIR} &&
                        cd ${TARGET_DIR} &&
                        npm install &&
                        nohup node index.js > app.log 2>&1 &
                    "
                    '''
                }
            }
        }


        stage('Deploy to Docker VM') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'mykey',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh '''
                    ssh -i "$SSH_KEY" -o StrictHostKeyChecking=no ${SSH_USER}@docker << 'EOF'
                    rm -rf /home/laborant/sample-node-app
                    git clone https://github.com/kere199/devops_class_exam.git /home/laborant/sample-node-app
                    cd /home/laborant/sample-node-app
                    docker build -t sample-node-app:latest .
                    docker rm -f sample-node || true
                    docker run -d -p 4444:4444 --name sample-node sample-node-app:latest
                    EOF
                    '''
                }
            }
        }



        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([
                    credentialsId: 'myapikey',
                    serverUrl: "${KUBE_API}"
                ]) {
                    sh '''
                      kubectl apply -f k8s/deployment.yaml
                      kubectl apply -f k8s/service.yaml
                    '''
                }
            }
        }
    }
}
