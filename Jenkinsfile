pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Remove existing image if it exists to avoid conflicts
                    sh 'docker rmi -f my-python-app || true'

                    // Build new image
                    sh 'docker build -t my-python-app .'
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    sh '''
                    docker run --rm -v "$PWD:/app" my-python-app \
                    pytest --junitxml=/app/test-results/results.xml --disable-warnings || true
                    '''
                }
            }
        }

        stage('Deploy') {
           steps {
            withCredentials([usernamePassword(
            credentialsId: 'docker-hub-creds',
            passwordVariable: 'DOCKER_PASSWORD',
            usernameVariable: 'DOCKER_USERNAME'
        )]) {
            script {
                sh '''
                echo "Logging into Docker Hub..."
                echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin

                echo "Tagging image..."
                docker tag my-python-app:latest $DOCKER_USERNAME/my-python-app:latest

                echo "Pushing image..."
                docker push $DOCKER_USERNAME/my-python-app:latest
                '''
            }
        }
    }
}

    }

    post {
        always {
            script {
                if (fileExists('test-results/results.xml')) {
                    junit 'test-results/results.xml'
                } else {
                    echo "No test results found to archive."
                }

                // Clean up dangling images to avoid disk space issues
                sh 'docker image prune -f || true'
            }

            // Archive test result files
            archiveArtifacts artifacts: '**/test-results/*.xml', allowEmptyArchive: true
        }
    }
}