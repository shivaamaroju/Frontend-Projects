pipeline {
    agent any

    environment {
        DOCKER_USER = "shivaamaroju"
        IMAGE_NAME = "3d-social-icons"
        SUB_DIR = "3d social media icons" 
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
                // This command tells K3s to apply your deployment and service
                // It will automatically update the pods with the 'latest' image we just pushed
                sh "kubectl apply -f deployment.yaml"
                
                // Force a rollout restart to ensure the new 'latest' image is pulled
                sh "kubectl rollout restart deployment social-icons-deployment"
            }
        }
    }
}
