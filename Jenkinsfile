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

    stage('Push Docker image to Nexus') {
      environment {
        NEXUS_REGISTRY = 'nexus:5000'
        // Must match your Nexus docker (hosted) repository name; change if different.
        TARGET_IMAGE   = "nexus:5000/docker-hosted/demo:${BUILD_NUMBER}"
      }
      steps {
        withCredentials([usernamePassword(credentialsId: 'nexus-docker-creds',
                                          usernameVariable: 'NEXUS_USER',
                                          passwordVariable: 'NEXUS_PASS')]) {
          sh """
            set -e
            docker tag demo:${BUILD_NUMBER} ${TARGET_IMAGE}
            echo "\$NEXUS_PASS" | docker login ${NEXUS_REGISTRY} -u "\$NEXUS_USER" --password-stdin
            docker push ${TARGET_IMAGE}
            docker logout ${NEXUS_REGISTRY} || true
          """
        }
      }
    }
  }

  post { always { echo 'Pipeline finished.' } }
}

