pipeline {
  agent any
  environment {
    DOCKERHUB_CREDENTIALS = 'docker-hub-creds'  // Jenkins credential ID
    IMAGE_NAME = 'manobhiramreddy/demo-app'
    IMAGE_TAG = "${env.BUILD_NUMBER}"
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build (npm)') {
      steps {
        bat 'npm ci'           // on Windows agents you may use bat 'npm ci'
        bat 'npm run build'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          bat "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
          bat "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
        }
      }
    }

    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
          bat "echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin"
          bat "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
          bat "docker push ${IMAGE_NAME}:latest"
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        // Option A: using Kubernetes CLI plugin and a secret file credential (kubeconfig-demo)
        withKubeConfig([credentialsId: 'kubeconfig-demo']) {
          // update image on the existing deployment (rolling update)
          bat "kubectl set image deployment/demo-app demo-app=${IMAGE_NAME}:${IMAGE_TAG} --record || true"
          // if resource not present, create
          bat "kubectl apply -f k8s/deployment.yaml || true"
          bat "kubectl apply -f k8s/service.yaml || true"
        }
      }
    }
  }

  post {
    success {
      echo "Pipeline completed. Deployed ${IMAGE_NAME}:${IMAGE_TAG}"
    }
    failure {
      echo "Pipeline failed."
    }
  }
}
