pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                // Apply ansiColor to the steps within this stage.
                ansiColor('xterm') {
                    sh 'echo "Starting the build process..."'
                    sh 'echo "Simulating a successful build."'
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
}
