pipeline {
    agent any

    tools {
        jdk 'jdk17'            // configure in Jenkins Global Tools
        maven 'maven-3.9'      // configure in Jenkins Global Tools
    }

    environment {
        MAVEN_SETTINGS_FILE_ID = 'maven-settings-file-id'  // Jenkins Managed File ID
        SONARQUBE_SERVER = 'SonarQube'                    // Name configured in Jenkins
        TOMCAT_CREDS = 'tomcat-manager-creds'             // Jenkins credentials (username+password)
        TOMCAT_HOST = 'your-tomcat-server-ip'             // Change to your Tomcat server IP
        APP_CONTEXT = '/myapp'                            // Tomcat app context
    }

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '15'))
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Unit Test') {
            steps {
                configFileProvider([configFile(fileId: "${MAVEN_SETTINGS_FILE_ID}", variable: 'MAVEN_SETTINGS')]) {
                    sh 'mvn -B -s $MAVEN_SETTINGS clean verify'
                }
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                    archiveArtifacts artifacts: 'target/*.war', fingerprint: true
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                configFileProvider([configFile(fileId: "${MAVEN_SETTINGS_FILE_ID}", variable: 'MAVEN_SETTINGS')]) {
                    withSonarQubeEnv("${SONARQUBE_SERVER}") {
                        sh 'mvn -B -s $MAVEN_SETTINGS sonar:sonar'
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
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
                withCredentials([usernamePassword(credentialsId: "${TOMCAT_CREDS}", usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                    sh '''
                        WAR_FILE=$(ls target/*.war | head -n1)
                        if [ -z "$WAR_FILE" ]; then
                          echo "No WAR file found!" && exit 1
                        fi
                        echo "Deploying $WAR_FILE to Tomcat..."
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
            echo "✅ Pipeline completed successfully"
        }
        failure {
            echo "❌ Pipeline failed"
        }
        always {
            cleanWs()
        }
    }
}
