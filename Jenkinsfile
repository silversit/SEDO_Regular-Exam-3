pipeline {
  agent any
  // Keep it simple: only timestamps (built-in). Remove ansiColor & empty triggers.
  options { timestamps() }

  environment {
    DOTNET_CLI_TELEMETRY_OPTOUT = '1'
    REPORT_DIR = 'reports'
  }

  stages {
    stage('Checkout') {
      steps {
        // Works because your job is "Pipeline script from SCM"
        checkout scm
        bat 'if not exist %REPORT_DIR% mkdir %REPORT_DIR%'
      }
    }

    stage('Restore') {
      steps {
        bat 'dotnet --info'
        bat 'dotnet restore'
      }
    }

    stage('Build') {
      steps {
        bat 'dotnet build --configuration Release --no-restore'
      }
    }

    stage('Test') {
      steps {
        // Produce TRX test results into reports/
        bat '''
          dotnet test --configuration Release --no-build ^
            --logger "trx;LogFileName=test-results.trx" ^
            --results-directory %REPORT_DIR%
        '''
        // Convert TRX -> JUnit XML so Jenkins can render test results
        bat '''
          dotnet tool update -g trx2junit
          set "PATH=%PATH%;%USERPROFILE%\\.dotnet\\tools"
          trx2junit %REPORT_DIR%\\*.trx
        '''
      }
      post {
        always {
          // Publish JUnit XML if present, and archive all reports
          junit allowEmptyResults: true, testResults: 'reports\\**\\*.xml'
          archiveArtifacts artifacts: 'reports\\**', allowEmptyArchive: true
        }
      }
    }
  }

  post {
    success { echo '✅ .NET build & tests passed.' }
    failure { echo '❌ Build or tests failed.' }
  }
}
