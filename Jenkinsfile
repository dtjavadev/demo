pipeline {
  agent any

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

    stage('Build') {
      steps {
        sh 'chmod +x ./gradlew'
        sh './gradlew --no-daemon clean bootJar'
      }
    }

    stage('Test') {
      steps {
        sh './gradlew --no-daemon test'
      }
    }

    stage('Docker Build') {
      steps {
        sh 'docker build -t demo:$BUILD_NUMBER .'
      }
    }
  }

  post { always { echo 'Pipeline finished.' } }
}

