pipeline {
    agent any 
    
    environment {
        // Define the virtual environment path
        VENV_PATH = 'venv'
    }
    
    stages {
        stage('Clone Repository') {
            steps {
                // Clean workspace and clone the repository
                cleanWs()
                git branch: 'main',
                    url: 'https://github.com/NikhilCyberk/simple_flask_app.git'
            }
        }
        
        stage('Set Up Python Environment') {
            steps {
                script {
                    // Create and activate virtual environment, install dependencies
                    bat """
                        python -m venv ${VENV_PATH}
                        ${VENV_PATH}\\Scripts\\activate.bat && python -m pip install --upgrade pip
                        ${VENV_PATH}\\Scripts\\activate.bat && pip install -r requirements.txt
                        ${VENV_PATH}\\Scripts\\activate.bat && pip install eventlet
                    """
                }
            }
        }
        
        stage('Run Server') {
            steps {
                script {
                    // Start Gunicorn with eventlet worker
                    bat """
                        ${VENV_PATH}\\Scripts\\activate.bat && python -m gunicorn --worker-class eventlet -w 1 -b 127.0.0.1:8000 app:app &
                        timeout /t 5 /nobreak > nul
                    """
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    try {
                        // Use powershell to make HTTP request since curl might not be available
                        bat '''
                            powershell -command "Invoke-WebRequest -Uri http://127.0.0.1:8000 -UseBasicParsing"
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
                    bat "${VENV_PATH}\\Scripts\\activate.bat && pytest"
                }
            }
        }
        
        stage('Post-Deployment Checks') {
            steps {
                script {
                    // Additional health checks
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
                // Cleanup: Kill Python processes
                bat 'taskkill /F /IM python.exe || exit 0'
                bat 'taskkill /F /IM gunicorn.exe || exit 0'
            }
        }
    }
}