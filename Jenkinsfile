pipeline {
    agent any
    environment {
        CONTROL_PLANE_IPS = '192.168.56.11 192.168.56.12 192.168.56.13'  // Control plane IPs
        SSH_USER = 'vagrant'
        SSH_KEY_PATH = '/var/lib/jenkins/.ssh/id_rsa'
        DOCKER_IMAGE = 'vinoji2005/train-schedule-app'
    }
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/vinoji2005/cicd-pipeline-train-schedule-autodeploy.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }
        stage('Push Docker Image') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub-credentials', url: '']) {
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    CONTROL_PLANE_IPS.split(' ').each { ip ->
                        sh """
                        ssh -i $SSH_KEY_PATH $SSH_USER@$ip 'kubectl apply -f deployment.yaml'
                        ssh -i $SSH_KEY_PATH $SSH_USER@$ip 'kubectl apply -f service.yaml'
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

