pipeline {
  agent any

  environment {
    SENTRIX_URL = "http://backend:8000"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Resolve Application') {
      steps {
        script {
          env.APP_ID = sh(
            script: """
              curl -s $SENTRIX_URL/api/appsec/applications |
              jq -r '.applications[] | select(.repo_url=="${env.GIT_URL}") | .id'
            """,
            returnStdout: true
          ).trim()

          if (!env.APP_ID) {
            error "Application not registered in Sentrix"
          }
        }
      }
    }

    stage('Trigger SAST') {
      steps {
        sh """
        curl -X POST $SENTRIX_URL/api/appsec/scans \
          -H "Content-Type: application/json" \
          -d '{
            "application_id": "$APP_ID",
            "scan_type": "sast",
            "branch": "main"
          }'
        """
      }
    }

    stage('Trigger SCA') {
      steps {
        sh """
        curl -X POST $SENTRIX_URL/api/appsec/scans \
          -H "Content-Type: application/json" \
          -d '{
            "application_id": "$APP_ID",
            "scan_type": "sca",
            "branch": "main"
          }'
        """
      }
    }
  }
}
