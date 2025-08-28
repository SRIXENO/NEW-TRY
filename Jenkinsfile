pipeline {
    agent any

    tools {
        jdk 'jdk17'        // Must match Jenkins configured JDK
        maven 'maven-3.9'  // Must match Jenkins configured Maven
    }

    environment {
        MVN_HOME = tool 'maven-3.9'
        NEXUS_REPO_ID = 'nexus'                // must match <id> in pom.xml
        NEXUS_URL = 'http://<NEXUS_IP>:8081'  // Nexus server URL
        MAVEN_SETTINGS = "$HOME/.m2/settings.xml"
        DEPLOY_ENV = 'staging'                // Change to production if needed
    }

    options {
        // Keep build logs for 30 days and discard older builds
        buildDiscarder(logRotator(daysToKeepStr: '30', numToKeepStr: '50'))
        timeout(time: 60, unit: 'MINUTES') // Pipeline timeout
        ansiColor('xterm')                  // Colorized logs
    }

    triggers {
        // Trigger on SCM change (push)
        pollSCM('H/5 * * * *') // every 5 mins
    }

    stages {

        stage('Checkout') {
            steps {
                echo "🔄 Checking out source code"
                checkout scm
            }
        }

        stage('Code Quality / Static Analysis') {
            steps {
                echo "🔍 Running static code analysis"
                // Example using Maven plugin like sonar
                sh 'mvn clean verify sonar:sonar -s $MAVEN_SETTINGS || true'
            }
        }

        stage('Unit Tests') {
            steps {
                echo "🧪 Running unit tests"
                sh 'mvn test -s $MAVEN_SETTINGS'
                junit '**/target/surefire-reports/*.xml'
            }
        }

        stage('Build Artifact') {
            steps {
                echo "🏗 Building the project"
                sh 'mvn clean package -s $MAVEN_SETTINGS'
            }
            post {
                success {
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }

        stage('Deploy to Nexus') {
            steps {
                echo "📦 Deploying artifact to Nexus"
                sh "mvn deploy -s $MAVEN_SETTINGS"
            }
        }

        stage('Deploy to Environment') {
            when {
                expression { return env.DEPLOY_ENV == 'staging' || env.DEPLOY_ENV == 'production' }
            }
            steps {
                echo "🚀 Deploying artifact to $DEPLOY_ENV environment"
                // Example: shell script to copy artifact to server
                // Replace with your actual deploy script
                sh """
                   scp target/*.jar user@staging-server:/opt/apps/
                   ssh user@staging-server 'systemctl restart my-app.service'
                """
            }
        }

    }

    post {
        always {
            echo "📌 Pipeline finished: ${currentBuild.currentResult}"
        }
        success {
            mail to: 'team@example.com',
                 subject: "✅ Build Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Build ${env.BUILD_NUMBER} was successful.\nCheck Jenkins for details: ${env.BUILD_URL}"
        }
        failure {
            mail to: 'team@example.com',
                 subject: "❌ Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Build ${env.BUILD_NUMBER} failed.\nCheck Jenkins for details: ${env.BUILD_URL}"
        }
    }
}
