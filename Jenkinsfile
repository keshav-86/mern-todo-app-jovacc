pipeline {
  agent any

  tools {
    nodejs 'Node18'
  }

  environment {
    SONAR_TOKEN = credentials('sonar-token')
    NEXUS_CREDS = credentials('nexus-creds')
    NEXUS_URL = "http://localhost:8081"
    NEXUS_REPO = "raw-release"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Install Backend Deps') {
      steps {
        dir('TODO/todo_backend') {
          bat 'npm ci'
        }
      }
    }

    stage('Install Frontend Deps & Build') {
      steps {
        dir('TODO/todo_frontend') {
          bat 'npm ci'
          bat 'npm run build'
        }
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('SonarQube') {
          bat """
            sonar-scanner ^
              -Dsonar.projectKey=mern-todo-app ^
              -Dsonar.sources=TODO/todo_backend,TODO/todo_frontend ^
              -Dsonar.host.url=http://localhost:9000 ^
              -Dsonar.login=${SONAR_TOKEN}
          """
        }
      }
    }

    stage('Package Artifacts') {
      steps {
        bat '''
          if exist artifacts rmdir /s /q artifacts
          mkdir artifacts
          powershell -Command "Compress-Archive -Path TODO\\todo_backend\\* -DestinationPath artifacts\\backend-%BUILD_NUMBER%.zip"
          powershell -Command "Compress-Archive -Path TODO\\todo_frontend\\build\\* -DestinationPath artifacts\\frontend-%BUILD_NUMBER%.zip"
        '''
        archiveArtifacts artifacts: 'artifacts/*', fingerprint: true
      }
    }

    stage('Upload to Nexus') {
      steps {
        bat """
          curl -v -u ${NEXUS_CREDS_USR}:${NEXUS_CREDS_PSW} --upload-file artifacts/backend-%BUILD_NUMBER%.zip ${NEXUS_URL}/repository/${NEXUS_REPO}/backend-%BUILD_NUMBER%.zip
          curl -v -u ${NEXUS_CREDS_USR}:${NEXUS_CREDS_PSW} --upload-file artifacts/frontend-%BUILD_NUMBER%.zip ${NEXUS_URL}/repository/${NEXUS_REPO}/frontend-%BUILD_NUMBER%.zip
        """
      }
    }
  }

  post {
    success { echo "✅ Pipeline completed successfully!" }
    failure { echo "❌ Pipeline failed!" }
  }
}
