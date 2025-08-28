pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Building project..."
                sh 'echo "Hello Jenkins Pipeline!"'
            }
        }

        stage('Test') {
            steps {
                echo "Running unit tests..."
                sh 'echo "Pretend we ran tests here"'
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying application..."
                sh 'echo "Pretend deployment to Tomcat here"'
            }
        }
    }
}
