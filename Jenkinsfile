pipeline {
    agent none  // Avoid assigning a global agent

    environment {
        DOCKER_IMAGE = "galshed1107/spam-detection:latest"  // Your DockerHub username
        REGISTRY_CREDENTIALS = credentials('docker')  // Credentials for DockerHub
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
            agent any  // Use any available agent for this stage
            steps {
                script {
                    echo "Running tests in Docker container"
                    powershell '''
                        docker run --rm -v $PWD:/app -w /app ${DOCKER_IMAGE} sh -c "pip install -r requirements.txt && pytest tests/"
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            agent any  // Use any available agent for this stage
            steps {
                script {
                    echo "Building Docker image"
                    powershell '''
                        docker build -t ${DOCKER_IMAGE} .
                    '''

                    echo "Pushing Docker image to DockerHub"
                    withDockerRegistry([url: 'https://index.docker.io/v1/', credentialsId: 'docker']) {
                        powershell '''
                            docker push ${DOCKER_IMAGE}
                        '''
                    }
                }
            }
        }

        stage('Deploy') {
            agent any  // Use any available agent for this stage
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
        // Automatically trigger the build when a change is pushed to the Git repository
        githubPush()  // This trigger listens for GitHub push events (you need to configure GitHub webhook)
    }
}
