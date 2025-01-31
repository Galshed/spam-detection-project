pipeline {
  agent none

  environment {
    DOCKER_IMAGE = "python:3.9"
    REGISTRY_CREDENTIALS = credentials('docker')
  }

  stages {
    stage('Checkout') {
      agent any  
      steps {
        script {
          echo "Cloning the repository"
          git branch: 'main', url: 'https://github.com/manuCprogramming/spam-detection-project.git'
        }
      }
    }

    stage('Build and Test') {
      agent {
        dockerImage "${DOCKER_IMAGE}"
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
        dockerImage "${DOCKER_IMAGE}"
      }
      steps {
        script {
          echo "Building Docker image"
          sh 'docker build -t ${DOCKER_IMAGE} .'

          echo "Pushing Docker image to registry"
          withDockerRegistry([url: 'https://index.docker.io/v1/', credentialsId: 'docker']) {
            sh 'docker push ${DOCKER_IMAGE}'
          }
        }
      }
    }

    stage('Deploy') {
      agent {
        dockerImage "${DOCKER_IMAGE}"
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
