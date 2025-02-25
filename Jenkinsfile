pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "rksavaliya/my-node-app:latest"
    }

    stages {
        stage('Cleanup Workspace') {
            steps {
                sh 'rm -rf *'  // Deletes everything in the workspace before cloning
            }
        }

        stage('Checkout Code') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-access', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    sh 'git clone https://$GIT_USERNAME:$GIT_PASSWORD@github.com/RaviSavaliya07/my-node-app.git .'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'docker-hub-token', variable: 'DOCKER_HUB_PASS')]) {
                    sh 'echo "$DOCKER_HUB_PASS" | docker login -u "rksavaliya" --password-stdin'
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                sh 'docker push $DOCKER_IMAGE'
            }
        }

        stage('Deploy on Local EC2') {
            steps {
                sh '''
                docker pull $DOCKER_IMAGE
                docker stop my-node-app || true
                docker rm my-node-app || true
                docker run -d -p 3000:3000 --name my-node-app $DOCKER_IMAGE
                '''
            }
        }
    }
}
