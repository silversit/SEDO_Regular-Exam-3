pipeline {
  agent any
  options { timestamps(); ansiColor('xterm') }

  triggers {
    // If you set a GitHub webhook, enable this in the job:
    // githubPush()
    // Or fallback polling:
    // pollSCM('H/5 * * * *')
  }

  environment {
    DOTNET_CLI_TELEMETRY_OPTOUT = '1'
    REPORT_DIR = 'reports'
  }

  stages {
    stage('Checkout (main only)') {
      when { branch 'main' }
      steps {
        // If you pasted the pipeline, we must manually checkout:
        checkout([
          $class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[
            url: 'https://github.com/silversit/SEDO_Regular-Exam-3.git'
            // If private: add credentialsId: 'YOUR_CREDS_ID'
          ]]
        ])
        bat 'if not exist %REPORT_DIR% mkdir %REPORT_DIR%'
      }
    }

    stage('.NET Restore/Build/Test') {
      when { branch 'main' }
      steps {
        bat '''
          dotnet --info
          dotnet restore
          dotnet build --configuration Release --no-restore

          dotnet test --configuration Release --no-build ^
            --logger "trx;LogFileName=test-results.trx" ^
            --results-directory %REPORT_DIR%

          dotnet tool install -g trx2junit
          set "PATH=%PATH%;%USERPROFILE%\\.dotnet\\tools"
          trx2junit %REPORT_DIR%\\*.trx
        '''
      }
      post {
        always {
          junit allowEmptyResults: true, testResults: 'reports\\**\\*.xml'
          archiveArtifacts artifacts: 'reports\\**', allowEmptyArchive: true
        }
      }
    }
  }

  post {
    success { echo '✅ .NET build & tests passed on main.' }
    failure { echo '❌ Build or tests failed.' }
  }
}
