pipeline {
    agent any
    environment {
        CONTROL_PLANE_IPS = '192.168.56.11'
        SSH_USER = 'vagrant'
        SSH_KEY_PATH = '/var/lib/jenkins/.ssh/id_rsa'
        DOCKER_IMAGE = 'vinoji2005/train-schedule-app'
    }
    options {
        ws('/var/lib/jenkins/workspace/Devops')
    }
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/vinoji2005/cicd.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                    echo "Building Docker Image: $DOCKER_IMAGE:${env.BUILD_ID}"
                    docker build -t $DOCKER_IMAGE:${env.BUILD_ID} .
                    """
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub-credentials', url: '']) {
                    sh """
                    echo "Pushing Docker Image to Docker Hub: $DOCKER_IMAGE:${env.BUILD_ID}"
                    docker push $DOCKER_IMAGE:${env.BUILD_ID}
                    """
                }
            }
        }
        stage('Prepare Kubernetes YAML') {
            steps {
                script {
                    sh """
                    echo "Preparing Kubernetes YAML with updated image name and tag"
                    sed -i 's|$DOCKER_IMAGE|'"$DOCKER_IMAGE"'|' deployment.yaml
                    sed -i 's|$BUILD_NUMBER|'"${env.BUILD_ID}"'|' deployment.yaml
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
            cleanWs()
        }
    }
}
