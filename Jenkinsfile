pipeline {
    agent any
    environment {
        CONTROL_PLANE_IPS = '192.168.56.11'  // Control plane IPs
        SSH_USER = 'vagrant'
        SSH_KEY_PATH = '/var/lib/jenkins/.ssh/id_rsa'
        DOCKER_IMAGE = 'vinoji2005/train-schedule-app'
    }
    options {
        ws('/var/lib/jenkins/workspace/Devops')  // Custom workspace
    }
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/vinoji2005/cicd.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:${env.BUILD_ID} .'  // Tag image with BUILD_ID
            }
        }
        stage('Push Docker Image') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub-credentials', url: '']) {
                    sh 'docker push $DOCKER_IMAGE:${env.BUILD_ID}'  // Push tagged image
                }
            }
        }
        stage('Prepare Kubernetes YAML') {
            steps {
                // Replace placeholders in deployment.yaml dynamically
                sh """
                sed -i 's|\\$DOCKER_IMAGE|$DOCKER_IMAGE|' deployment.yaml
                sed -i 's|\\$BUILD_NUMBER|${env.BUILD_ID}|' deployment.yaml
                """
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    CONTROL_PLANE_IPS.split(' ').each { ip ->
                        sh """
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
