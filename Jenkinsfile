pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "rksavaliya/my-node-app:latest"
        APP_SERVER_IP = "54.219.31.96"  // Replace with actual Application Server IP
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

        stage('Login to Docker Hub') {
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
                sshagent(['app-server-ssh']) {  // Using SSH to deploy on App Server
                    sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@$APP_SERVER_IP << 'EOF'
                    echo "Pulling latest Docker image..."
                    docker pull $DOCKER_IMAGE

                    echo "Stopping existing container (if running)..."
                    docker stop my-node-app || true
                    docker rm my-node-app || true

                    echo "Running new container..."
                    docker run -d -p 3000:3000 --name my-node-app $DOCKER_IMAGE

                    echo "Deployment completed successfully."
                    exit 0
                    EOF
                    '''
                }
            }
        }
    }
}
