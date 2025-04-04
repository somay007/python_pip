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

        stage('Install Python') {
            steps {
                bat 'choco install python --version=3.10 -y'
                bat 'set PATH=%PATH%;C:\\Python310\\Scripts;C:\\Python310\\'
            }
        }

        stage('Build and Package') {
            steps {
                bat 'pip install -r requirements.txt'
                bat 'python setup.py build'
                bat 'python setup.py bdist_wheel'
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat "az login --service-principal -u %AZURE_CLIENT_ID% -p %AZURE_CLIENT_SECRET% --tenant %AZURE_TENANT_ID%"
                     bat 'powershell Compress-Archive -Path dist/* -DestinationPath publish.zip -Force'
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
