pipeline {
  agent any

  tools {
    nodejs 'Node18'
    // Make sure Node18 is configured in Jenkins > Global Tool Configuration
  }

  environment {
    SONAR_TOKEN = credentials('sonar-project-token')   // use your credential ID from Jenkins
    NEXUS_CREDS = credentials('nexus-creds')           // must match ID in Jenkins
    NEXUS_URL = "http://localhost:8081"
    NEXUS_REPO = "raw-release"
  }

  stages {
    stage('Checkout') {
      steps {
        echo "📦 Checking out code from GitHub..."
        checkout scm
      }
    }

    stage('Install Backend Deps') {
      steps {
        dir('TODO/todo_backend') {
          echo "📦 Installing backend dependencies..."
          bat 'npm ci'
        }
      }
    }

    stage('Install Frontend Deps & Build') {
      steps {
        dir('TODO/todo_frontend') {
          echo "⚙️ Installing frontend dependencies and building app..."
          bat 'npm ci'
          bat 'npm run build'
        }
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('MySonarQubeServer') { // ✅ Use your SonarQube name from Manage Jenkins > System
          bat """
            ${tool('SonarScanner')}\\bin\\sonar-scanner.bat ^
              -Dsonar.projectKey=mern-todo-app ^
              -Dsonar.sources=TODO/todo_backend,TODO/todo_frontend ^
              -Dsonar.host.url=http://localhost:9000 ^
              -Dsonar.login=%SONAR_TOKEN%
          """
        }
      }
    }

    stage('Package Artifacts') {
      steps {
        echo "📦 Packaging backend and frontend..."
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
        echo "🚀 Uploading artifacts to Nexus Repository..."
        bat """
          curl -v -u %NEXUS_CREDS_USR%:%NEXUS_CREDS_PSW% --upload-file artifacts/backend-%BUILD_NUMBER%.zip %NEXUS_URL%/repository/%NEXUS_REPO%/backend-%BUILD_NUMBER%.zip
          curl -v -u %NEXUS_CREDS_USR%:%NEXUS_CREDS_PSW% --upload-file artifacts/frontend-%BUILD_NUMBER%.zip %NEXUS_URL%/repository/%NEXUS_REPO%/frontend-%BUILD_NUMBER%.zip
        """
      }
    }
  }

  post {
    success {
      echo "✅ Pipeline completed successfully!"
    }
    failure {
      echo "❌ Pipeline failed!"
    }
  }
}
