pipeline {
    agent any

    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal'
        RESOURCE_GROUP = 'rg-jenkins'
        APP_SERVICE_NAME = 'webapijenkins10101010'
        PYTHON_PATH = '"C:\Users\hp\AppData\Local\Programs\Python\Python313\python.exe"'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/somay007/python_pip.git'
            }
        }

        stage('Set Up Python Environment') {
            steps {
                bat '"%PYTHON_PATH%" -m venv venv'
                bat 'call venv\\Scripts\\activate && python -m pip install --upgrade pip'
                bat 'call venv\\Scripts\\activate && pip install -r requirements.txt'
                bat 'call venv\\Scripts\\activate && pip install pytest'
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                  bat 'echo Cleaning up existing publish folder...'
                  bat 'if exist publish (rmdir /s /q publish)'
                  bat 'mkdir publish'

                  bat 'echo Copying .py files to publish folder...'
                  bat 'for %%f in (*.py) do copy "%%f" publish\\'

                  bat 'echo Copying requirements.txt if it exists...'
                  bat 'if exist requirements.txt copy requirements.txt publish\\'


                    bat 'az login --service-principal --username %AZURE_CLIENT_ID% --password %AZURE_CLIENT_SECRET% --tenant %AZURE_TENANT_ID%'
                    
                    // Zip the publish folder properly
                    bat 'powershell Compress-Archive -Path publish\\* -DestinationPath publish.zip -Force'
                    
                    // Ensure publish.zip exists before deployment
                    bat 'if not exist publish.zip (echo "Error: publish.zip not created!" & exit 1)'

                    // Deploy the zip file
                    bat 'az webapp deploy --resource-group %RESOURCE_GROUP% --name %APP_SERVICE_NAME% --src-path "%CD%\\publish.zip" --type zip'
                }
            }
        }
    }

    post {
        failure {
            echo 'Deployment Failed! Check logs for errors.'
        }
        success {
            echo 'Deployment Successful!'
        }
    }
}
