pipeline {
    agent none  // Avoid assigning a global agent

    environment {
        DOCKER_IMAGE = "galshed1107/spam-detection:latest"  // Your DockerHub username
        REGISTRY_CREDENTIALS = credentials('docker')  // Credentials for DockerHub
    }

    stages {
        stage('Checkout') {
            agent any
            steps {
                script {
                    echo "Cloning the repository..."
                    checkout scm
                }
            }
        }

        stage('Build and Test') {
            agent any
            steps {
                script {
                    echo "Running tests in Docker container"
                    powershell '''
                        $currentDir = (Get-Location).Path
                        docker run --rm -v "$currentDir:/app" -w /app $env:DOCKER_IMAGE sh -c "pip install -r requirements.txt && pytest tests/"
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            agent any
            steps {
                script {
                    echo "Building Docker image"
                    powershell '''
                        docker build -t $env:DOCKER_IMAGE .
                    '''

                    echo "Pushing Docker image to DockerHub"
                    withDockerRegistry([url: 'https://index.docker.io/v1/', credentialsId: 'docker']) {
                        powershell '''
                            docker push $env:DOCKER_IMAGE
                        '''
                    }
                }
            }
        }

        stage('Deploy') {
            agent any
            steps {
                script {
                    echo "Deploying to Kubernetes"
                    powershell '''
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                    '''
                }
            }
        }
    }

    triggers {
        githubPush()
    }
}
