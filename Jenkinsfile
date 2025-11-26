pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub')
        BACKEND_IMAGE = 'pushpendrachaudhari/backendapp:latest'
        FRONTEND_IMAGE = 'pushpendrachaudhari/frontendapp:latest'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/PUSHPENDRACHAUDHARI/devops-automation-project.git'
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                dir('backend') {
                    sh 'docker build -t backend-app:dev .'
                    sh "docker tag backend-app:dev ${env.BACKEND_IMAGE}"
                }
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                dir('frontend') {
                    sh 'docker build -t frontend-app:dev .'
                    sh "docker tag frontend-app:dev ${env.FRONTEND_IMAGE}"
                }
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                script {
                    // Login using Docker Hub PAT
                    sh "echo ${DOCKER_HUB_CREDENTIALS_PSW} | docker login -u ${DOCKER_HUB_CREDENTIALS_USR} --password-stdin"
                    sh "docker push ${env.BACKEND_IMAGE}"
                    sh "docker push ${env.FRONTEND_IMAGE}"
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@15.206.164.69 "
                    docker pull pushpendrachaudhari/backendapp:latest
                    docker pull pushpendrachaudhari/frontendapp:latest
                    docker stop backend || true
                    docker stop frontend || true
                    docker rm backend || true
                    docker rm frontend || true
                    docker run -d --name backend -p 5000:5000 pushpendrachaudhari/backendapp:latest
                    docker run -d --name frontend -p 80:80 pushpendrachaudhari/frontendapp:latest
                    "
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "CI/CD Pipeline Completed"
        }
    }
}

