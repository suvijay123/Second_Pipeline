pipeline {
    agent any

    environment {
        APP_NAME = "pollpoint"
        PORT = "8000"
    }

    stages {
        stage('Checkout') {
            steps {
                // Pull source code from SCM repository
                checkout scm
            }
        }

        stage('Validate Application') {
            steps {
                // Verify essential files exist before proceeding
                sh 'test -f app.py'
                sh 'test -f requirements.txt'
                echo 'Application validation successful!'
            }
        }

        stage('Build Docker Image') {
            steps {
                // Build the Docker image for PollPoint
                sh "docker build -t ${APP_NAME}:latest ."
            }
        }

        stage('Deploy Container') {
            steps {
                // Stop existing container, remove it, run the new one with a persistent data volume
                sh """
                    docker stop ${APP_NAME} || true
                    docker rm ${APP_NAME} || true
                    
                    docker volume create pollpoint-data || true

                    docker run -d \\
                        --name ${APP_NAME} \\
                        -p ${PORT}:8000 \\
                        -v pollpoint-data:/data \\
                        ${APP_NAME}:latest
                """
            }
        }

        stage('Health Check') {
            steps {
                // Verify the app is alive and responding
                sh '''
                    sleep 3
                    curl -f http://localhost:8000/health || exit 1
                    echo "PollPoint is live and healthy!"
                '''
            }
        }
    }

    post {
        always {
            echo "Pipeline run completed."
        }
    }
}
