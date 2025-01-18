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
                    // Split the environment setup into separate commands for better error tracking
                    bat 'python -m venv venv'
                    bat 'venv\\Scripts\\activate.bat && python -m pip install --upgrade pip'
                    
                    // Install each dependency individually with verbose output
                    bat '''
                        venv\\Scripts\\activate.bat && (
                            pip install Flask==3.0.0 --verbose
                            pip install pytest==7.4.3 --verbose
                            pip install gunicorn==21.2.0 --verbose
                            pip install eventlet==0.33.3 --verbose
                            pip install Werkzeug==3.0.1 --verbose
                        )
                    '''
                    
                    // Verify installations
                    bat 'venv\\Scripts\\activate.bat && pip list'
                }
            }
        }
        
        // stage('Run Server') {
        //     steps {
        //         script {
        //             bat '''
        //                 venv\\Scripts\\activate.bat && (
        //                     echo Starting Flask application...
        //                     python app.py
        //                 )
        //             '''
        //         }
        //     }
        // }
        stage('Run Server') {
    steps {
        script {
            bat '''
                venv\\Scripts\\activate.bat && (
                    echo Starting Flask application...
                    start /B gunicorn --bind 127.0.0.1:8000 app:app
                    timeout /t 5 /nobreak > nul
                )
            '''
        }
    }
}
        
        stage('Verify Deployment') {
            steps {
                script {
                    try {
                        bat '''
                            timeout /t 5 /nobreak > nul
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
                    bat 'venv\\Scripts\\activate.bat && python -m pytest'
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
                bat 'taskkill /F /IM python.exe || exit 0'
            }
        }
        failure {
            script {
                echo 'Pipeline failed! Checking virtual environment status...'
                bat '''
                    echo Listing installed packages:
                    venv\\Scripts\\activate.bat && pip list
                    echo Checking Python version:
                    python --version
                '''
            }
        }
    }
}