pipeline {
    agent any
    environment {
        CONTROL_PLANE_IPS = '192.168.56.11'  // Control plane IPs
        SSH_USER = 'vagrant'
        SSH_KEY_PATH = '/var/lib/jenkins/.ssh/id_rsa'
        DOCKER_IMAGE_NAME = 'vinoji2005/train-schedule-app'  // Docker Hub image name
    }
    options {
        ws('/var/lib/jenkins/workspace/Devops')  // Custom workspace
    }
    stages {
        stage('Clone Repository') {
            steps {
                // Clone the repository from GitHub
                git branch: 'main', url: 'https://github.com/vinoji2005/cicd.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                    echo "Building Docker Image: $DOCKER_IMAGE_NAME:${env.BUILD_ID}"
                    docker build -t $DOCKER_IMAGE_NAME:${env.BUILD_ID} .
                    """
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub-credentials', url: '']) {
                    sh """
                    echo "Pushing Docker Image to Docker Hub: $DOCKER_IMAGE_NAME:${env.BUILD_ID}"
                    docker push $DOCKER_IMAGE_NAME:${env.BUILD_ID}
                    """
                }
            }
        }
        stage('Prepare Kubernetes YAML') {
            steps {
                script {
                    sh """
                    echo "Preparing Kubernetes YAML with updated image name and tag"
                    echo "Before Replacement:"
                    cat deployment.yaml
                    sed -i 's|\\$DOCKER_IMAGE_NAME|'"$DOCKER_IMAGE_NAME"'|' deployment.yaml
                    sed -i 's|\\$BUILD_NUMBER|'"${env.BUILD_ID}"'|' deployment.yaml
                    echo "After Replacement:"
                    cat deployment.yaml
                    """
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    CONTROL_PLANE_IPS.split(' ').each { ip ->
                        sh """
                        echo "Deploying to Kubernetes Control Plane: $ip"
                        scp -i $SSH_KEY_PATH deployment.yaml $SSH_USER@$ip:/home/$SSH_USER/
                        ssh -i $SSH_KEY_PATH $SSH_USER@$ip 'kubectl apply -f /home/$SSH_USER/deployment.yaml'
                        """
                    }
                }
            }
        }
    }
    post {
        always {
            // Clean up the workspace to avoid conflicts
            cleanWs()
        }
    }
}
