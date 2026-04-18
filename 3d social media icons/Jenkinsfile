pipeline {
    agent any

    environment {
        DOCKER_USER = "shivaamaroju"
        IMAGE_NAME = "3d-social-icons"
        CONTAINER_NAME = "social-icons-app"
        // Target the specific sub-folder in your repo
        SUB_DIR = "3d social media icons" 
    }

    stages {
        stage('Checkout') {
            steps {
                // Pulls the whole 'Frontend-Projects' repo
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
                // IMPORTANT: Move into the sub-folder where the code is
                dir("${env.SUB_DIR}") {
                    sh "docker build --no-cache -t ${env.DOCKER_USER}/${env.IMAGE_NAME}:${BUILD_NUMBER} ."
                    sh "docker tag ${env.DOCKER_USER}/${env.IMAGE_NAME}:${BUILD_NUMBER} ${env.DOCKER_USER}/${env.IMAGE_NAME}:latest"
                    sh "docker push ${env.DOCKER_USER}/${env.IMAGE_NAME}:${BUILD_NUMBER}"
                    sh "docker push ${env.DOCKER_USER}/${env.IMAGE_NAME}:latest"
                }
            }
        }

        stage('Clean Old & Run New') {
            steps {
                sh "docker stop ${env.CONTAINER_NAME} || true"
                sh "docker rm ${env.CONTAINER_NAME} || true"
                // Running on port 8091 so it doesn't clash with your old project
                sh "docker run -d --name ${env.CONTAINER_NAME} -p 8090:80 ${env.DOCKER_USER}/${env.IMAGE_NAME}:latest"
            }
        }
    }

    post {
        success {
            sh 'docker image prune -f'
            echo "Successfully deployed to http://<your-vm-ip>:8090"
        }
    }
}
