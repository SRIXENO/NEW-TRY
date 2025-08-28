pipeline {
    agent any

    tools {
        jdk 'jdk17'                // Jenkins Global Tool Config: name=jdk17
        maven 'maven-3.9'          // Jenkins Global Tool Config: name=maven-3.9
    }

    environment {
        MAVEN_SETTINGS_FILE_ID = 'maven-settings-xml'     // ID from Jenkins Managed Files
        SONARQUBE_SERVER = 'SonarQube'                   // Name from Jenkins SonarQube config
        TOMCAT_CREDS = 'tomcat-manager-creds'            // Jenkins credentials (username+password)
        TOMCAT_HOST = '192.168.1.100'                    // Replace with your Tomcat server IP
        APP_CONTEXT = '/myapp'                           // Tomcat app context path
    }

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '15'))
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "📥 Checking out source code from Git..."
                checkout scm
            }
        }

        stage('Build & Unit Test') {
            steps {
                echo "⚙️ Running Maven build and unit tests..."
                configFileProvider([configFile(fileId: "${MAVEN_SETTINGS_FILE_ID}", variable: 'MAVEN_SETTINGS')]) {
                    sh 'mvn -B -s $MAVEN_SETTINGS clean verify'
                }
            }
            post {
                always {
                    echo "📂 Archiving reports and artifacts..."
                    junit 'target/surefire-reports/*.xml'
                    archiveArtifacts artifacts: 'target/*.war', fingerprint: true
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "🔍 Running SonarQube static analysis..."
                configFileProvider([configFile(fileId: "${MAVEN_SETTINGS_FILE_ID}", variable: 'MAVEN_SETTINGS')]) {
                    withSonarQubeEnv("${SONARQUBE_SERVER}") {
                        sh 'mvn -B -s $MAVEN_SETTINGS sonar:sonar'
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                echo "✅ Checking SonarQube Quality Gate..."
                script {
                    def qg = waitForQualityGate()
                    if (qg.status != 'OK') {
                        error "Pipeline failed due to SonarQube Quality Gate: ${qg.status}"
                    }
                }
            }
        }

        stage('Publish to Nexus') {
            steps {
                echo "📦 Publishing artifacts to Nexus repository..."
                configFileProvider([configFile(fileId: "${MAVEN_SETTINGS_FILE_ID}", variable: 'MAVEN_SETTINGS')]) {
                    sh 'mvn -B -s $MAVEN_SETTINGS deploy -DskipTests'
                }
            }
        }

        stage('Deploy to Tomcat') {
            when {
                branch 'main'
            }
            steps {
                echo "🚀 Deploying application to Tomcat server..."
                withCredentials([usernamePassword(credentialsId: "${TOMCAT_CREDS}", usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                    sh '''
                        WAR_FILE=$(ls target/*.war | head -n1)
                        if [ -z "$WAR_FILE" ]; then
                          echo "❌ No WAR file found!" && exit 1
                        fi
                        echo "Deploying $WAR_FILE to Tomcat at ${TOMCAT_HOST}..."
                        curl --fail -u $TOMCAT_USER:$TOMCAT_PASS \
                          --upload-file "$WAR_FILE" \
                          "http://${TOMCAT_HOST}:8080/manager/text/deploy?path=${APP_CONTEXT}&update=true"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline completed successfully"
        }
        failure {
            echo "❌ Pipeline failed"
        }
        always {
            echo "🧹 Cleaning workspace..."
            cleanWs()
        }
    }
}
