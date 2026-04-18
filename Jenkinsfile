pipeline {
    agent any

    environment {
        DOCKER_USER = "shivaamaroju"
        IMAGE_NAME = "3d-social-icons"
        SUB_DIR = "3d social media icons" 
        // This tells kubectl to use the K3s security credentials
        KUBECONFIG = "/etc/rancher/k3s/k3s.yaml"
    }

    stages {
        stage('Checkout & Login') {
            steps {
                git branch: 'master', url: 'https://github.com/shivaamaroju/Frontend-Projects.git'
                withCredentials([usernamePassword(credentialsId: 'docker-hub-token', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                }
            }
        }

        stage('Build & Push') {
            steps {
                dir("${env.SUB_DIR}") {
                    sh "docker build --no-cache -t ${env.DOCKER_USER}/${env.IMAGE_NAME}:${BUILD_NUMBER} ."
                    sh "docker tag ${env.DOCKER_USER}/${env.IMAGE_NAME}:${BUILD_NUMBER} ${env.DOCKER_USER}/${env.IMAGE_NAME}:latest"
                    sh "docker push ${env.DOCKER_USER}/${env.IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy to K3s') {
            steps {
                // Apply the deployment and force a restart to pull the newest 'latest' image
                sh "kubectl apply -f deployment.yaml --validate=false"
                sh "kubectl rollout restart deployment social-icons-deployment"
            }
        }
    }

    post {
        success {
            sh 'docker image prune -f'
            echo "Successfully deployed to K3s! Check http://<your-vm-ip>:30007"
        }
    }
}
