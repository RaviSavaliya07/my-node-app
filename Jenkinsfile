pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "rksavaliya/my-node-app:latest"
        APP_SERVER_IP = "54.219.31.96"
    }

    stages {
        stage('Cleanup Workspace') {
            steps {
                deleteDir()
            }
        }

        stage('Checkout Code') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-access', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    sh 'git clone https://$GIT_USERNAME:$GIT_PASSWORD@github.com/RaviSavaliya07/my-node-app.git .'
                }
            }
        }

        stage('Login to Docker Hub') {  // NEW: Ensure authentication before building
            steps {
                withCredentials([string(credentialsId: 'docker-hub-token', variable: 'DOCKER_HUB_PASS')]) {
                    sh 'echo "$DOCKER_HUB_PASS" | docker login -u "rksavaliya" --password-stdin'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'docker-hub-token', variable: 'DOCKER_HUB_PASS')]) {
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }

        stage('Deploy to Application Server') {
            steps {
                sshagent(['app-server-ssh']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@$APP_SERVER_IP <<EOF
                    docker pull $DOCKER_IMAGE
                    docker stop my-node-app || true
                    docker rm my-node-app || true
                    docker run -d -p 3000:3000 --name my-node-app $DOCKER_IMAGE
                    EOF
                    '''
                }
            }
        }
    }
}
