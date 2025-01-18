pipeline {
    agent any 
    
    environment {
        VENV_PATH = 'venv'
    }
    
    stages {
        stage('Clone Repository') {
            steps {
                cleanWs()
                git branch: 'main',
                    url: 'https://github.com/NikhilCyberk/simple_flask_app.git'
            }
        }
        
        stage('Set Up Python Environment') {
            steps {
                script {
                    // Create virtual environment, upgrade pip, and install dependencies with verbose output
                    bat """
                        python -m venv ${VENV_PATH}
                        ${VENV_PATH}\\Scripts\\activate.bat && python -m pip install --upgrade pip
                        ${VENV_PATH}\\Scripts\\activate.bat && pip install -r requirements.txt --verbose
                        ${VENV_PATH}\\Scripts\\activate.bat && pip list
                    """
                }
            }
        }
        
        stage('Run Server') {
            steps {
                script {
                    bat """
                        ${VENV_PATH}\\Scripts\\activate.bat && python app.py &
                        timeout /t 5 /nobreak > nul
                    """
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    try {
                        bat '''
                            powershell -command "Start-Sleep -s 3; Invoke-WebRequest -Uri http://127.0.0.1:8000 -UseBasicParsing"
                        '''
                        echo 'Application is running successfully!'
                    } catch (Exception e) {
                        error 'Failed to verify deployment: Application is not responding'
                    }
                }
            }
        }
        
        stage('Run Unit Tests') {
            steps {
                script {
                    bat """
                        ${VENV_PATH}\\Scripts\\activate.bat && python -m pytest
                    """
                }
            }
        }
        
        stage('Post-Deployment Checks') {
            steps {
                script {
                    bat '''
                        powershell -command "Invoke-WebRequest -Uri http://127.0.0.1:8000/health -UseBasicParsing"
                    '''
                }
            }
        }
    }
    
    post {
        always {
            script {
                bat '''
                    taskkill /F /IM python.exe || exit 0
                '''
            }
        }
        failure {
            script {
                echo 'Pipeline failed! Check the pip installation logs and Python environment setup.'
            }
        }
    }
}