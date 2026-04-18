pipeline {
    agent any

    environment {
        DOCKER_USER = "shivaamaroju"
        IMAGE_NAME = "3d-social-icons"
        SUB_DIR = "3d social media icons" 
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/shivaamaroju/Frontend-Projects.git'
            }
        }

        stage('Login to DockerHub') {
            steps {
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

        stage('Deploy to MicroK8s') {
            steps {
                // MicroK8s uses its own kubectl command. 
                // We use --validate=false to skip openapi authentication errors.
                sh "microk8s kubectl apply -f deployment.yaml --validate=false"
                
                // Force Kubernetes to pull the 'latest' image we just pushed
                sh "microk8s kubectl rollout restart deployment social-icons-deployment"
            }
        }
    }

    post {
        success {
            sh 'docker image prune -f'
            echo "Successfully deployed to MicroK8s!"
            echo "Check your External IP: microk8s kubectl get svc"
        }
        failure {
            echo "Deployment failed. Check if 'jenkins' user is in 'microk8s' group."
        }
    }
}
