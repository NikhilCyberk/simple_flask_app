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
                    bat 'python -m venv venv'
                    bat 'venv\\Scripts\\activate.bat && python -m pip install --upgrade pip'
                    bat '''
                        venv\\Scripts\\activate.bat && (
                            pip install Flask==3.0.0 --verbose
                            pip install pytest==7.4.3 --verbose
                            pip install waitress==2.1.2 --verbose
                            pip install Werkzeug==3.0.1 --verbose
                        )
                    '''
                }
            }
        }
        
        stage('Run Server') {
            steps {
                script {
                    // Create a batch file to run the server using waitress instead of gunicorn
                    bat '''
                        echo from waitress import serve > serve.py
                        echo from app import app >> serve.py
                        echo serve(app, host='127.0.0.1', port=8000) >> serve.py
                        
                        echo @echo off > run_server.bat
                        echo venv\\Scripts\\activate.bat >> run_server.bat
                        echo start /B python serve.py >> run_server.bat
                        start /B run_server.bat
                        ping 127.0.0.1 -n 6 > nul
                    '''
                }
            }
        }
        
        stage('Run Unit Tests') {
            steps {
                script {
                    bat 'venv\\Scripts\\activate.bat && python -m pytest'
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    bat '''
                        powershell -Command "try { $response = Invoke-WebRequest -Uri http://127.0.0.1:8000 -UseBasicParsing; exit 0 } catch { exit 1 }"
                    '''
                    echo 'Application is running successfully!'
                }
            }
        }
        
        stage('Post-Deployment Checks') {
            steps {
                script {
                    bat '''
                        powershell -Command "try { $response = Invoke-WebRequest -Uri http://127.0.0.1:8000/health -UseBasicParsing; exit 0 } catch { exit 1 }"
                    '''
                }
            }
        }
    }
    
    post {
        always {
            script {
                bat '''
                    taskkill /F /IM python.exe > nul 2>&1 || exit 0
                '''
            }
        }
        failure {
            script {
                echo 'Pipeline failed! Checking virtual environment status...'
                bat '''
                    echo Listing installed packages:
                    venv\\Scripts\\activate.bat && pip list
                '''
            }
        }
    }
}