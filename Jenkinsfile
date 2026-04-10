def appImage

pipeline {
  agent any

  options {
    timestamps()
    ansiColor('xterm')
  }

  environment {
    APP_NAME = 'demo'
    IMAGE_TAG = "${APP_NAME}:${env.BUILD_NUMBER}"
    CONTAINER_NAME = 'demo-app'
    APP_PORT = '8080'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Test (Gradle)') {
      steps {
        script {
          docker.image('gradle:8.7-jdk17').inside {
            sh 'chmod +x ./gradlew || true'
            sh './gradlew --no-daemon clean test'
          }
        }
      }
    }

    stage('Build Docker image') {
      steps {
        script {
          appImage = docker.build(env.IMAGE_TAG)
        }
      }
    }

    stage('Run Docker container') {
      steps {
        script {
          // Keep a single stable container name per environment.
          // We still shell out for cleanup because the Docker Pipeline API does not expose
          // "remove container by name" helpers directly.
          sh """
            set -eux
            if docker ps -a --format '{{.Names}}' | grep -qx '${env.CONTAINER_NAME}'; then
              docker rm -f '${env.CONTAINER_NAME}'
            fi
          """

          // Leaves the container running after the build (same behavior as your original pipeline).
          appImage.run("--name ${env.CONTAINER_NAME} -p ${env.APP_PORT}:8080")
        }
      }
    }
  }

  post {
    always {
      echo "Image: ${env.IMAGE_TAG}, container: ${env.CONTAINER_NAME}, port: ${env.APP_PORT}"
    }
  }
}

