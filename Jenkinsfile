pipeline {
    agent any

    tools {
        jdk 'jdk'
        maven 'maven'
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
                withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_API_KEY')]) {
                    sh '''
                        mvn org.owasp:dependency-check-maven:check \
                            -DnvdApiKey=$NVD_API_KEY \
                            -DskipTests \
                            -Dformat=ALL \
                            -DoutputDirectory=target/dependency-check-report
                    '''
                }
            }
        }
    }

    post {
        success {
            script {
                def timestamp = sh(
                    script: "date +'%Y-%m-%d %H:%M:%S'",
                    returnStdout: true
                ).trim()

                // Safe Slack notification using shell interpolation
                withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_WEBHOOK_URL')]) {
                    sh '''
                        PAYLOAD=$(printf '{"text":"✅ BUILD SUCCESS:\\n• Job: %s\\n• Build Number: #%s\\n• Triggered By: Jenkins\\n• Time (IST): %s\\n• Build URL: %s\\n• Report Path: %s/target/dependency-check-report"}' \
                            "$JOB_NAME" "$BUILD_NUMBER" "$timestamp" "$BUILD_URL" "$WORKSPACE")
                        curl -s -X POST -H "Content-Type: application/json" --data "$PAYLOAD" "$SLACK_WEBHOOK_URL"
                    '''
                }

                // Email notification (Groovy interpolation is safe here since no secret is included)
                mail to: 'ps191701@gmail.com',
                     subject: "✅ BUILD SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: """✅ BUILD SUCCESS
• Job: ${env.JOB_NAME}
• Build Number: #${env.BUILD_NUMBER}
• Triggered By: Jenkins
• Time (IST): ${timestamp}
• Build URL: ${env.BUILD_URL}
• Report Path: ${env.WORKSPACE}/target/dependency-check-report
"""
            }
        }

        failure {
            script {
                def timestamp = sh(
                    script: "date +'%Y-%m-%d %H:%M:%S'",
                    returnStdout: true
                ).trim()

                withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_WEBHOOK_URL')]) {
                    sh '''
                        PAYLOAD=$(printf '{"text":"❌ BUILD FAILURE:\\n• Job: %s\\n• Build Number: #%s\\n• Triggered By: Jenkins\\n• Time (IST): %s\\n• Build URL: %s"}' \
                            "$JOB_NAME" "$BUILD_NUMBER" "$timestamp" "$BUILD_URL")
                        curl -s -X POST -H "Content-Type: application/json" --data "$PAYLOAD" "$SLACK_WEBHOOK_URL"
                    '''
                }

                mail to: 'ps191701@gmail.com',
                     subject: "❌ BUILD FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: """❌ BUILD FAILURE
• Job: ${env.JOB_NAME}
• Build Number: #${env.BUILD_NUMBER}
• Triggered By: Jenkins
• Time (IST): ${timestamp}
• Build URL: ${env.BUILD_URL}
"""
            }
        }

        always {
            echo "Archiving Dependency Check Reports..."
            archiveArtifacts artifacts: 'target/dependency-check-report/**', allowEmptyArchive: true
        }
    }
}
