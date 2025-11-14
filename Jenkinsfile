pipeline {
  agent any

  environment {
    DOCKERHUB_REPO = 'najeebpatoana/docker-jenkins'
    IMAGE_TAG = "${env.BUILD_NUMBER}"
  }

  options { 
    timestamps() 
  }

  stages {
    stage('Checkout') {
      steps {
        echo 'Pulling latest code from GitHub...'
        checkout scm
      }
    }

    stage('Build Docker Image') {
      steps {
        echo 'Building Docker image...'
        bat """
          docker build -t ${DOCKERHUB_REPO}:${IMAGE_TAG} .
          docker tag ${DOCKERHUB_REPO}:${IMAGE_TAG} ${DOCKERHUB_REPO}:latest
        """
      }
    }

    stage('Push to Docker Hub') {
      steps {
        echo 'Pushing image to Docker Hub...'
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          bat """
            echo %PASS% | docker login -u %USER% --password-stdin
            docker push ${DOCKERHUB_REPO}:${IMAGE_TAG}
            docker push ${DOCKERHUB_REPO}:latest
          """
        }
      }
    }

    stage('Deploy') {
      steps {
        echo 'Deploying new version...'
        bat """
          docker stop docker_jenkins 2>nul || echo Container not running
          docker rm docker_jenkins 2>nul || echo Container not found
          docker run -d --name docker_jenkins -p 8081:3000 ${DOCKERHUB_REPO}:${IMAGE_TAG}
        """
      }
    }
  }

  post {
    success {
      echo "SUCCESS! Build #${env.BUILD_NUMBER}"
      echo "App running at: http://localhost:8081"
    }
    failure {
      echo " Build #${env.BUILD_NUMBER} failed!"
    }
    always {
      bat 'docker image prune -f'
    }
  }
}