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
                    bat 'venv\\Scripts\\activate && pip install -r requirements.txt'
                }
            }
        }
        stage('Run Gunicorn') {
            steps {
                script {
                    bat 'venv\\Scripts\\activate && gunicorn -b 127.0.0.1:8000 app:app'
                }
            }
        }
        stage('Verify Deployment') {
            steps {
                script {
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
}