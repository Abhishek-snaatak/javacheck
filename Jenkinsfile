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
                // Correctly pass the API key to Maven
                sh '''#!/bin/bash
                    mvn org.owasp:dependency-check-maven:check \
                        -DnvdApiKey="${NVD_API_KEY}" \
                        -DskipTests \
                        -Dformat=ALL \
                        -DoutputDirectory="target/dependency-check-report"
                '''
            }
        }
    }

    post {
        success {
            script {
                def timestamp = sh(script: "date +'%Y-%m-%d %H:%M:%S'", returnStdout: true).trim()
                def msg = """✅ BUILD SUCCESS:
• Job: ${env.JOB_NAME}
• Build Number: #${env.BUILD_NUMBER}
• Time (IST): ${timestamp}
• Build URL: ${env.BUILD_URL}
• Report Path: ${WORKSPACE}/target/dependency-check-report
"""
                // Slack notification
                sh """#!/bin/bash
                    curl -s -X POST \
                        -H 'Content-Type: application/json' \
                        --data "{\\"text\\": \\"${msg}\\":}" \
                        "${SLACK_WEBHOOK_URL}"
                """
                // Email notification
                mail to: 'ps191701@gmail.com',
                     subject: "✅ BUILD SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: msg
            }
        }

        failure {
            script {
                def timestamp = sh(script: "date +'%Y-%m-%d %H:%M:%S'", returnStdout: true).trim()
                def msg = """❌ BUILD FAILURE:
• Job: ${env.JOB_NAME}
• Build Number: #${env.BUILD_NUMBER}
• Time (IST): ${timestamp}
• Build URL: ${env.BUILD_URL}
"""
                sh """#!/bin/bash
                    curl -s -X POST \
                        -H 'Content-Type: application/json' \
                        --data "{\\"text\\": \\"${msg}\\"}" \
                        "${SLACK_WEBHOOK_URL}"
                """
                mail to: 'ps191701@gmail.com',
                     subject: "❌ BUILD FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: msg
            }
        }

        always {
            echo "Archiving Dependency Check Reports..."
            archiveArtifacts artifacts: 'target/dependency-check-report/**', allowEmptyArchive: true
        }
    }
}
