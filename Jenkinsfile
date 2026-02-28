pipeline {
  agent any

  environment {
    // Must match: Manage Jenkins → System → SonarQube servers → Name
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
          python3 --version
          python3 -m venv .venv
          . .venv/bin/activate
          pip install -r app/requirements.txt
          pytest -q
        '''
      }
    }

    stage('SonarQube Scan') {
      steps {
        withSonarQubeEnv("${SONARQUBE_SERVER}") {
          script {
            // Must match: Manage Jenkins → Tools → SonarQube Scanner installations → Name
            def scannerHome = tool('sonar-scanner')

            // Add scanner binary to PATH for this block
            withEnv(["PATH+SONAR=${scannerHome}/bin"]) {
              sh '''
                set -e
                echo "Running SonarQube scan..."
                sonar-scanner
              '''
            }
          }
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