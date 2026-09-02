
pipeline {

    agent any

    environment {
        APP_SERVER = '172.31.41.57'
        APP_USER = 'ubuntu'
        APP_DIR = '/opt/hello-springboot'
        APP_NAME = 'hello-springboot'

        SSH_CREDENTIALS = 'prod-server-ssh'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'

                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Building Spring Boot application...'

                sh '''
                    chmod +x mvnw 2>/dev/null || true

                    ./mvnw clean package -DskipTests
                '''
            }
        }

        stage('Push Artifact to Nexus') {
            steps {
                echo 'Uploading artifact to Nexus...'

                sh '''
                    chmod +x mvnw 2>/dev/null || true

                    ./mvnw deploy -DskipTests
                '''
            }
        }

        stage('Deploy to Application Server') {
            steps {

                sshagent(credentials: [env.SSH_CREDENTIALS]) {

                    sh '''
                        set -e

                        echo "======================================"
                        echo "Testing SSH connection"
                        echo "======================================"

                        ssh -o BatchMode=yes \
                            -o StrictHostKeyChecking=no \
                            ${APP_USER}@${APP_SERVER} \
                            "whoami"

                        echo "======================================"
                        echo "Creating application directory"
                        echo "======================================"

                        ssh -o BatchMode=yes \
                            -o StrictHostKeyChecking=no \
                            ${APP_USER}@${APP_SERVER} \
                            "mkdir -p ${APP_DIR}"

                        echo "======================================"
                        echo "Copying artifact to application server"
                        echo "======================================"

                        scp -o StrictHostKeyChecking=no \
                            target/*.jar \
                            ${APP_USER}@${APP_SERVER}:${APP_DIR}/app.jar

                        echo "======================================"
                        echo "Restarting application"
                        echo "======================================"

                        ssh -o BatchMode=yes \
                            -o StrictHostKeyChecking=no \
                            ${APP_USER}@${APP_SERVER} \
                            "sudo systemctl restart ${APP_NAME}"

                        echo "======================================"
                        echo "Checking application status"
                        echo "======================================"

                        ssh -o BatchMode=yes \
                            -o StrictHostKeyChecking=no \
                            ${APP_USER}@${APP_SERVER} \
                            "sudo systemctl is-active ${APP_NAME}"

                        echo "======================================"
                        echo "Deployment completed successfully"
                        echo "======================================"
                    '''
                }
            }
        }
    }

    post {

        success {
            echo 'BUILD + NEXUS + DEPLOYMENT SUCCESSFUL'
        }

        failure {
            echo 'BUILD / NEXUS / DEPLOYMENT FAILED'
        }
    }
}

