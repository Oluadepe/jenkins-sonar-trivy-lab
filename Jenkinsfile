pipeline {
  agent any

  environment {
    // Matches your screenshot!
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
          python3 -m venv .venv
          . .venv/bin/activate
          pip install -r app/requirements.txt
          python3 -m pytest -q
        '''
      }
    }

    stage('SonarQube Scan') {
      steps {
        // This looks up the tool path manually to bypass the "Invalid tool type" error
        script {
            def scannerHome = tool 'sonar-scanner'
            withSonarQubeEnv("${SONARQUBE_SERVER}") {
                sh "${scannerHome}/bin/sonar-scanner"
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
        sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
      }
    }

    stage('Trivy Scan (Image)') {
      steps {
        sh '''
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