pipeline {
  agent { label 'docker-cloud' }

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

    stage('Run (smoke)') {
      steps {
        sh '''
          set -eu
          JAR="$(ls -1 build/libs/*.jar | grep -v -- '-plain.jar' | head -n 1)"
          echo "Running: $JAR --help"
          java -jar "$JAR" --help >/dev/null
        '''
      }
    }
  }

  post { always { echo 'Pipeline finished.' } }
}

