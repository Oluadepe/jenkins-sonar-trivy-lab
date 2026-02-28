pipeline {
  agent any

  environment {
    SONARQUBE_SERVER = 'SonarQube-Server'
    IMAGE_NAME = 'jenkins-sonar-trivy-lab'
    IMAGE_TAG  = "${env.BUILD_NUMBER}"
  }

  options {
    timestamps()
    disableConcurrentBuilds()
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
          set -e
          python --version
          python -m venv .venv
          . .venv/bin/activate
          pip install -r app/requirements.txt
          pytest -q
        '''
      }
    }

    stage('SonarQube Scan') {
      steps {
        withSonarQubeEnv("${SONARQUBE_SERVER}") {
          sh '''
            set -e
            echo "Running SonarQube scan..."

            SCANNER_HOME="$(tool 'sonar-scanner')"
            export PATH="$SCANNER_HOME/bin:$PATH"

            sonar-scanner
          '''
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 10, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        sh '''
          set -e
          echo "Building docker image..."
          docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
        '''
      }
    }

    stage('Trivy Scan (Image)') {
      steps {
        sh '''
          set -e
          echo "Running Trivy scan via Docker (no install needed)..."
          docker run --rm \
            -v /var/run/docker.sock:/var/run/docker.sock \
            aquasec/trivy:latest \
            image --exit-code 1 --severity HIGH,CRITICAL ${IMAGE_NAME}:${IMAGE_TAG}
        '''
      }
    }
  }

  post {
    always {
      echo "Result: ${currentBuild.currentResult}"
    }
  }
}