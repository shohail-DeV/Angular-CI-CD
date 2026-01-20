pipeline {
    agent any

    tools {
        nodejs 'node20'
    }

    environment {
        APP_NAME = "angular-app"
        IIS_PATH = "C:\\inetpub\\angular-app"
        DIST_PATH = "dist\\test\\browser"
        BACKUP_PATH = "C:\\inetpub\\_backups"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://your-git-repo-url.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npm ci'
            }
        }

        stage('Build Angular (Browser Only)') {
            steps {
                bat 'npx ng build --configuration production'
            }
        }

        stage('Validate Build Output') {
            steps {
                bat """
                if not exist %DIST_PATH% (
                  echo Build output missing
                  exit /b 1
                )
                """
            }
        }

        stage('Backup Current IIS Deployment') {
            steps {
                bat """
                if not exist %BACKUP_PATH% mkdir %BACKUP_PATH%
                if exist %IIS_PATH% (
                  xcopy %IIS_PATH% %BACKUP_PATH%\\%APP_NAME%_%BUILD_NUMBER% /E /I /Y
                )
                """
            }
        }

        stage('Deploy to IIS') {
            steps {
                bat """
                for %%f in (%IIS_PATH%\\*) do (
                  if /I not "%%~nxf"=="web.config" del /Q "%%f"
                )
                xcopy %DIST_PATH% %IIS_PATH% /E /I /Y
                """
            }
        }

        stage('Smoke Test') {
            steps {
                powershell '''
                try {
                  $r = Invoke-WebRequest http://localhost:8099 -UseBasicParsing -TimeoutSec 10
                  if ($r.StatusCode -ne 200) { throw "Health check failed" }
                } catch {
                  throw "Deployment validation failed"
                }
                '''
            }
        }
    }

    post {
        success {
            echo "Deployment successful"
        }
        failure {
            echo "Deployment failed — backup available for rollback"
        }
    }
}
