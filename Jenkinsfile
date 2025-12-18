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
                      mvn -DskipTests \
                          org.owasp:dependency-check-maven:check \
                          -DnvdApiKey=$NVD_API_KEY \
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
                def msg = """✅ BUILD SUCCESS: 
• Job: ${env.JOB_NAME}
• Build Number: #${env.BUILD_NUMBER}
• Triggered By: Jenkins
• Time (IST): ${timestamp}
• Build URL: ${env.BUILD_URL}
• Report Path: ${WORKSPACE}/target/dependency-check-report
"""
                withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_WEBHOOK_URL')]) {
                    def payload = groovy.json.JsonOutput.toJson([text: msg])
                    withEnv(["PAYLOAD=${payload}"]) {
                        sh '''
                          curl -s -X POST \
                            -H "Content-Type: application/json" \
                            --data "$PAYLOAD" \
                            "$SLACK_WEBHOOK_URL"
                        '''
                    }
                }
                mail to: 'ps191701@gmail.com',
                     subject: "✅ BUILD SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: msg
            }
        }
        failure {
            script {
                def timestamp = sh(
                    script: "date +'%Y-%m-%d %H:%M:%S'",
                    returnStdout: true
                ).trim()
                def msg = """❌ BUILD FAILURE
• Job: ${env.JOB_NAME}
• Build Number: #${env.BUILD_NUMBER}
• Triggered By: Jenkins
• Time (IST): ${timestamp}
• Build URL: ${env.BUILD_URL}
"""
                withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_WEBHOOK_URL')]) {
                    def payload = groovy.json.JsonOutput.toJson([text: msg])
                    withEnv(["PAYLOAD=${payload}"]) {
                        sh '''
                          curl -s -X POST \
                            -H "Content-Type: application/json" \
                            --data "$PAYLOAD" \
                            "$SLACK_WEBHOOK_URL"
                        '''
                    }
                }
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
