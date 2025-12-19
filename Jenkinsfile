pipeline {
    agent any

    environment {
        GIT_REPO = "https://github.com/kere199/devops_class_exam.git"

        TARGET_HOST = "172.16.0.3"
        TARGET_DIR  = "/home/laborant/sample-node-app"

        DOCKER_HOST = "docker"
        DOCKER_APP  = "/home/laborant/sample-node-app"

        KUBE_API = "https://kubernetes:6443"
        DOCKER_IMAGE = "yourdockerusername/sample-node-app:latest"
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
                    sh """
                    ssh -i "\$SSH_KEY" -o StrictHostKeyChecking=no \$SSH_USER@\$TARGET_HOST < /dev/null << 'ENDSSH'
                        rm -rf ${TARGET_DIR}
                        git clone ${GIT_REPO} ${TARGET_DIR}
                        cd ${TARGET_DIR}
                        npm install
                        nohup node index.js > app.log 2>&1 < /dev/null &
                    ENDSSH
                    """
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
                    sh """
                    ssh -i "\$SSH_KEY" -o StrictHostKeyChecking=no \$SSH_USER@\$DOCKER_HOST < /dev/null << 'ENDSSH'
                        docker pull ${DOCKER_IMAGE}

                        docker stop nodejs-app || true
                        docker rm nodejs-app || true

                        docker run -d --name nodejs-app -p 4444:4444 ${DOCKER_IMAGE}

                        sleep 3
                        curl -f http://localhost:4444 && echo "Docker VM deployment successful!"
                    ENDSSH
                    """
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
