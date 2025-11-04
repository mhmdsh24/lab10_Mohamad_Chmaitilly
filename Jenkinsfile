pipeline {
    agent any
    environment {
        VIRTUAL_ENV = 'venv'
    }
    stages {
        stage('Setup') {
            steps {
                script {
                    if (!fileExists("${env.WORKSPACE}/${VIRTUAL_ENV}")) {
                        bat "\"C:\\Users\\LENOVO\\AppData\\Local\\Programs\\Python\\Python313\\python.exe\" -m venv ${VIRTUAL_ENV}"
                    }
                    bat "call ${VIRTUAL_ENV}\\Scripts\\activate.bat && pip install -r requirements.txt"
                }
            }
        }
        stage('Lint') {
            steps {
                script {
                    bat "call ${VIRTUAL_ENV}\\Scripts\\activate.bat && flake8 app.py"
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    bat "call ${VIRTUAL_ENV}\\Scripts\\activate.bat && set PYTHONPATH=%CD% && pytest"
                }
            }
        }
        stage('Coverage') {
            steps {
                script {
                    bat """
                        call ${VIRTUAL_ENV}\\Scripts\\activate.bat && \
                        set PYTHONPATH=%CD% && \
                        coverage run -m pytest && \
                        coverage report && \
                        coverage html
                    """
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'htmlcov/**', allowEmptyArchive: true
                }
            }
        }
        stage('Security Scan') {
            steps {
                script {
                    bat """
                        call ${VIRTUAL_ENV}\\Scripts\\activate.bat && bandit -r . -f json -o bandit-report.json || (echo Bandit scan completed with warnings or errors & ver > nul) && bandit -r . -f txt -o bandit-report.txt || (echo Bandit scan completed with warnings or errors & ver > nul)
                    """
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'bandit-report.*', allowEmptyArchive: true
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    bat """
                        call ${VIRTUAL_ENV}\\Scripts\\activate.bat && \
                        echo Deploying application... && \
                        if not exist deploy (mkdir deploy) && \
                        copy app.py deploy\\ && \
                        copy requirements.txt deploy\\ && \
                        echo Application files copied to deploy directory
                    """
                    // For remote deployment, uncomment and configure:
                    // bat """
                    //     scp -r deploy\\* user@remote-server:/path/to/deployment/
                    //     ssh user@remote-server "cd /path/to/deployment && pip install -r requirements.txt"
                    // """
                    echo "Deployment completed successfully!"
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
