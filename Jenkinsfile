pipeline {
    agent none

    environment {
        DOCKER_IMAGE = "galshed1107/spam-detection:latest"
        REGISTRY_CREDENTIALS = credentials('docker')
    }

    stages {
        stage('Checkout') {
            agent any
            steps {
                checkout scm
            }
        }

        stage('Build and Test') {
            agent any
            steps {
                script {
                    echo "Running tests in Docker container"

                    // Use forward slashes for path consistency
                    def currentDir = pwd().replace('\\', '/')

                    // Improved Docker command with correct variable substitution
                    sh """
                        docker run --rm -v "${currentDir}:/app" -w /app ${DOCKER_IMAGE} pip install -r requirements.txt && pytest tests/
                    """
                }
            }
        }

        stage('Build Docker Image') {
            agent any
            steps {
                script {
                    echo "Building Docker image"
                    sh "docker build -t ${DOCKER_IMAGE} ."

                    echo "Pushing Docker image to DockerHub"
                    withDockerRegistry([url: 'https://index.docker.io/v1/', credentialsId: 'docker']) {
                        sh "docker push ${DOCKER_IMAGE}"
                    }
                }
            }
        }

        stage('Deploy') {
            agent any
            steps {
                script {
                    echo "Deploying to Kubernetes"
                    sh """
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                    """
                }
            }
        }
    }

    triggers {
        githubPush()
    }
}
