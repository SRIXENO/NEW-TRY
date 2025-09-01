pipeline {
    agent any
    stages {
        stage('Build') {
    steps {
        sh 'docker build -t $REPO_NAME:$IMAGE_TAG .'
                }
            }
        }
        stage('Push') {
            steps {
                ansiColor('xterm') {
                    sh 'echo "Simulating a push to the container registry."'
                }
            }
        }
        stage('Deploy') {
            steps {
                ansiColor('xterm') {
                    sh 'echo "Simulating deployment to the t2.micro instance."'
                }
            }
        }
    }
