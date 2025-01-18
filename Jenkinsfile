pipeline {
    agent any 
    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/NikhilCyberk/simple_flask_app.git'
            }
        }
        stage('Set Up Python Environment') {
            steps {
                script {
                    bat 'python -m venv venv' // Create virtual environment
                    bat 'venv\\Scripts\\activate' // Activate virtual environment
                    bat 'pip install -r requirements.txt' // Install dependencies
                }
            }
        }
        stage('Run Gunicorn') {
            steps {
                script {
                    bat 'venv\\Scripts\\activate && gunicorn -b 127.0.0.1:8000 app:app' // Start Gunicorn server
                }
            }
        }
        stage('Verify Deployment') {
            steps {
                script {
                    bat 'curl http://127.0.0.1:8000' // Verify deployment
                }
            }
        }
        stage('Run Unit Tests') {
            steps {
                script {
                    bat 'venv\\Scripts\\activate && pytest' // Run unit tests
                }
            }
        }
        stage('Post-Deployment Checks') {
            steps {
                script {
                    // Add any endpoint checks or additional verification here
                    echo 'Post-deployment checks completed.'
                }
            }
        }
    }
}
