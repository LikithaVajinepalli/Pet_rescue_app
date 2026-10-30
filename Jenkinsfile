
pipeline {
    agent any

    stages {
        stage('Build Docker Image') {
            steps {
                bat 'docker build -t petrescue:v1 .'
            }
        }

        stage('Run Selenium Tests') {
            steps {
                echo 'Run Selenium Tests'

                bat 'python -m pip install -r requirements.txt'

                bat 'start /B python app.py'

                bat 'timeout /t 5 /nobreak'

                bat 'python -m pytest -v'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    bat '''
                    @echo off
                    echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                bat 'docker tag petrescue:v1 likithavajinepalli/pet-rescue-app:new'

                bat 'docker push likithavajinepalli/pet-rescue-app:new'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat 'kubectl apply -f deployment.yaml --validate=false'

                bat 'kubectl apply -f service.yaml'
            }
        }
    }

    post {
        always {
            bat '''
            @echo off
            echo Cleaning up application process...
            taskkill /F /IM python.exe /T 2>NUL
            exit /b 0
            '''
        }

        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
    }
}
