pipeline {
    agent none  // Avoid assigning a global agent

    environment {
        DOCKER_IMAGE = "python:3.9"
        REGISTRY_CREDENTIALS = credentials('docker')
    }

    stages {
        stage('Checkout') {
            agent any  // Use any available agent for checkout
            steps {
                script {
                    echo "Cloning the repository..."
                    checkout scm  // Use SCM plugin to clone the repo
                }
            }
        }

        stage('Build and Test') {
            agent {
                docker {
                    image "${DOCKER_IMAGE}"
                    args '-v $PWD:/app -w /app'  // Mount workspace
                }
            }
            steps {
                script {
                    echo "Installing dependencies"
                    sh 'pip install -r requirements.txt'

                    echo "Running tests"
                    sh 'pytest tests/'
                }
            }
        }

        stage('Build Docker Image') {
            agent {
                label 'docker'  // Run this step on a node with Docker installed
            }
            steps {
                script {
                    echo "Building Docker image"
                    sh 'docker build -t myrepo/spam-detection:latest .'

                    echo "Pushing Docker image to registry"
                    withDockerRegistry([url: 'https://index.docker.io/v1/', credentialsId: 'docker']) {
                        sh 'docker push myrepo/spam-detection:latest'
                    }
                }
            }
        }

        stage('Deploy') {
            agent {
                label 'docker'  // Use a node with Docker installed for deployment
            }
            steps {
                script {
                    echo "Deploying to Kubernetes"
                    sh 'kubectl apply -f k8s/deployment.yaml'
                    sh 'kubectl apply -f k8s/service.yaml'
                }
            }
        }
    }
}
