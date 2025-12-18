pipeline {
    agent any

    tools {
        jdk 'jdk'
        maven 'maven'
    }

    environment {
        NVD_API_KEY = credentials('nvd-api-key')
        SLACK_WEBHOOK_URL = credentials('slack-webhook')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/OT-MICROSERVICES/salary-api'
            }
        }

        stage('Dependency Scan') {
            steps {
                sh '''#!/bin/bash
                    # Use a fresh data directory to prevent DB lock issues
                    mvn org.owasp:dependency-check-maven:purge \
                        -DdataDirectory="${WORKSPACE}/dependency-check-data"
                    
                    mvn org.owasp:dependency-check-maven:check \
                        -DnvdApiKey="${NVD_API_KEY}" \
                        -DskipTests \
                        -Dformat=ALL \
                        -DdataDirectory="${WORKSPACE}/dependency-check-data" \
                        -DoutputDirectory="${WORKSPACE}/target/dependency-check-report"
                '''
            }
        }
    }

    post {
        always {
            echo "Archiving Dependency Check Reports..."
            archiveArtifacts artifacts: 'target/dependency-check-report/**', allowEmptyArchive: true
        }
    }
}
