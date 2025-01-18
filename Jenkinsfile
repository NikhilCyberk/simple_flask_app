pipeline {
    agent any 
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/NikhilCyberk/simple_flask_app.git'
            }
        }
        stage('Set Up Python Environment') {
            steps {
                script {
                    bat 'python -m venv venv'
                    bat 'venv\\Scripts\\activate && pip install -r requirements.txt && pip install waitress'
                }
            }
        }
        stage('Run Server') {
            steps {
                script {
                    // Use waitress instead of gunicorn
                    bat '''
                        venv\\Scripts\\activate && python -c "from waitress import serve; from app import app; serve(app, host='127.0.0.1', port=8000)"
                    '''
                }
            }
        }
        stage('Verify Deployment') {
            steps {
                script {
                    // Add a small delay to ensure server is up
                    bat 'timeout /t 5'
                    bat 'curl http://127.0.0.1:8000'
                }
            }
        }
        stage('Run Unit Tests') {
            steps {
                script {
                    bat 'venv\\Scripts\\activate && pytest'
                }
            }
        }
        stage('Post-Deployment Checks') {
            steps {
                script {
                    echo 'Post-deployment checks completed.'
                }
            }
        }
    }
    post {
        always {
            script {
                // Cleanup: Kill any remaining Python processes
                bat 'taskkill /F /IM python.exe || exit 0'
            }
        }
    }
}