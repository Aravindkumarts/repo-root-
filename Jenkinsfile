pipeline {
  agent any

  environment {
    IMAGE_NAME = "local-ci-app"
    PORT = "8081"
  }

  options {
    timestamps()
    ansiColor('xterm')
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install & Test') {
      steps {
        sh '''
          cd app
          npm ci
          npm test
        '''
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          def img = docker.build("${IMAGE_NAME}:${env.BUILD_NUMBER}")
        }
      }
    }

    stage('Run Container') {
      steps {
        sh '''
          set -e
          # Stop/remove previous container if exists
          docker rm -f ${IMAGE_NAME} || true

          # Run new container
          docker run -d --name ${IMAGE_NAME} -p ${PORT}:3000 ${IMAGE_NAME}:${BUILD_NUMBER}
        '''
      }
    }
  }

  post {
    success {
      echo "Deployed at http://localhost:${PORT}"
    }
    failure {
      sh 'docker logs ${IMAGE_NAME} || true'
    }
    always {
      sh 'docker images | head -n 20'
    }
  }
}
