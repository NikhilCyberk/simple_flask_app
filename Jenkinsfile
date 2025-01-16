pipeline {
    agent any
    
    environment {
        PYTHON_ENV = 'venv'
        FLASK_PORT = '8000'
        APP_DIR = 'flask_app'
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Clean workspace before starting
                cleanWs()
                
                // Clone the repository
                checkout scm
                
                // Create application directory
                bat "mkdir %APP_DIR%"
            }
        }
        
        stage('Setup Python Environment') {
            steps {
                bat """
                    cd %APP_DIR%
                    
                    // Create virtual environment
                    python -m venv %PYTHON_ENV%
                    
                    // Activate virtual environment and install dependencies
                    call %PYTHON_ENV%\\Scripts\\activate.bat
                    python -m pip install --upgrade pip
                    pip install -r requirements.txt
                """
            }
        }
        
        stage('Static Code Analysis') {
            steps {
                bat """
                    cd %APP_DIR%
                    call %PYTHON_ENV%\\Scripts\\activate.bat
                    pip install flake8
                    flake8 . --exclude=venv/* --max-line-length=120
                """
            }
        }
        
        stage('Run Unit Tests') {
            steps {
                bat """
                    cd %APP_DIR%
                    call %PYTHON_ENV%\\Scripts\\activate.bat
                    pytest tests/ --junitxml=test-results/junit.xml -v
                """
            }
            post {
                always {
                    junit '%APP_DIR%/test-results/junit.xml'
                }
            }
        }
        
        stage('Deploy Application') {
            steps {
                bat """
                    cd %APP_DIR%
                    call %PYTHON_ENV%\\Scripts\\activate.bat
                    
                    // Kill any existing Gunicorn processes
                    taskkill /F /IM gunicorn.exe /T 2>NUL || exit /b 0
                    
                    // Start Gunicorn in the background using config file
                    start /B cmd /c "gunicorn -c gunicorn.conf.py app:app"
                    
                    // Give Gunicorn time to start
                    timeout /t 10 /nobreak
                """
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    // Multiple health checks
                    def maxRetries = 3
                    def retryCount = 0
                    def deployed = false
                    
                    while (!deployed && retryCount < maxRetries) {
                        try {
                            def response = bat(
                                script: "curl -s -o NUL -w \"%%{http_code}\" http://127.0.0.1:%FLASK_PORT%/health",
                                returnStdout: true
                            ).trim()
                            
                            if (response == "200") {
                                deployed = true
                                echo "Application successfully deployed and verified!"
                            } else {
                                error "Health check failed with status: ${response}"
                            }
                        } catch (Exception e) {
                            retryCount++
                            if (retryCount < maxRetries) {
                                echo "Retry ${retryCount}/${maxRetries} after 5 seconds..."
                                sleep(5)
                            } else {
                                error "Failed to verify deployment after ${maxRetries} attempts"
                            }
                        }
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo "Pipeline completed successfully! Application is running at http://127.0.0.1:${FLASK_PORT}"
        }
        failure {
            // Clean up on failure
            bat """
                cd %APP_DIR%
                taskkill /F /IM gunicorn.exe /T 2>NUL || exit /b 0
            """
        }
        cleanup {
            // Clean up workspace
            cleanWs()
        }
    }
}