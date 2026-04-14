pipeline {
  agent {
    docker {
      image 'jenkins/inbound-agent:latest-alpine-jdk17'
      args '-u 0:0'
      reuseNode true
    }
  }

  options {
    timestamps()
    ansiColor('xterm')
  }

  environment {
    APP_NAME = 'demo'
    IMAGE_NAME = "demo:${env.BUILD_NUMBER}"
    CONTAINER_NAME = 'demo-app'
    APP_PORT = '9090'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build (Gradle)') {
      steps {
        sh 'chmod +x ./gradlew'
        sh './gradlew --no-daemon clean test bootJar'
      }
    }

    stage('Build Docker image') {
      steps {
        sh 'docker build -t "$IMAGE_NAME" .'
      }
    }

    stage('Run Docker container') {
      steps {
        sh '''
          set -eux

          if docker ps -a --format '{{.Names}}' | grep -qx "$CONTAINER_NAME"; then
            docker rm -f "$CONTAINER_NAME"
          fi

          docker run -d --name "$CONTAINER_NAME" -p "$APP_PORT:$APP_PORT" "$IMAGE_NAME"
        '''
      }
    }
  }

  post {
    always {
      echo "Image: ${env.IMAGE_NAME}, container: ${env.CONTAINER_NAME}"
    }
  }
}

