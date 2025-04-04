pipeline {
    agent any
    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal'
        RESOURCE_GROUP = 'rg-jenkins'
        APP_SERVICE_NAME = 'pythonappjenkins10101010'
        PYTHON_INSTALLER_URL = 'https://www.python.org/ftp/python/3.13.0/python-3.13.0-amd64.exe'
        PYTHON_INSTALL_PATH = 'C:\\Python313'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/somay007/python_pip.git'
            }
        }

        stage('Install Python if not present') {
            steps {
                script {
                    def pythonExists = bat(script: 'where python', returnStatus: true) == 0
                    if (!pythonExists) {
                        echo 'Python not found, installing...'
                        bat "curl -o python-installer.exe %PYTHON_INSTALLER_URL%"
                        bat "start /wait python-installer.exe /quiet InstallAllUsers=1 PrependPath=1 TargetDir=%PYTHON_INSTALL_PATH%"
                        bat "setx PATH \"%PYTHON_INSTALL_PATH%;%PYTHON_INSTALL_PATH%\\Scripts;%PATH%\""
                        bat "del python-installer.exe"
                    } else {
                        echo 'Python is already installed.'
                    }
                }
            }
        }

        stage('Setup Python Environment') {
            steps {
                bat "%PYTHON_INSTALL_PATH%\\python --version"
                bat "%PYTHON_INSTALL_PATH%\\python -m venv venv"
                bat "call venv\\Scripts\\activate"
                bat "pip install --upgrade pip"
                bat "pip install -r requirements.txt"
            }
        }

        stage('Package Application') {
            steps {
                bat "powershell Compress-Archive -Path .\* -DestinationPath .\app.zip -Force"
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat "az login --service-principal -u %AZURE_CLIENT_ID% -p %AZURE_CLIENT_SECRET% --tenant %AZURE_TENANT_ID%"
                    bat "az webapp deploy --resource-group %RESOURCE_GROUP% --name %APP_SERVICE_NAME% --src-path .\app.zip --type zip"
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Deployment Failed!'
        }
    }
}
