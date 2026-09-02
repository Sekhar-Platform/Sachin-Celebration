pipeline {

    agent any

    environment {
        APP_SERVER = '172.31.41.57'
        APP_USER = 'ubuntu'
        APP_DIR = '/opt/sachin-celebration'
        APP_NAME = 'sachin-celebration'

        SSH_CREDENTIALS = 'prod-ssh-key'
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
                    set -e

                    echo "======================================"
                    echo "Building Spring Boot application"
                    echo "======================================"

                    mvn clean package -DskipTests

                    echo "======================================"
                    echo "Build completed successfully"
                    echo "======================================"

                    ls -lh target/*.jar
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
                        echo "Preparing deployment directory"
                        echo "======================================"

                        ssh -o BatchMode=yes \
                            -o StrictHostKeyChecking=no \
                            ${APP_USER}@${APP_SERVER} \
                            "sudo mkdir -p ${APP_DIR} && sudo chown ${APP_USER}:${APP_USER} ${APP_DIR}"

                        echo "======================================"
                        echo "Deployment directory ready"
                        echo "======================================"

                        ssh -o BatchMode=yes \
                            -o StrictHostKeyChecking=no \
                            ${APP_USER}@${APP_SERVER} \
                            "ls -ld ${APP_DIR}"

                        echo "======================================"
                        echo "Copying JAR to application server"
                        echo "======================================"

                        scp -o BatchMode=yes \
                            -o StrictHostKeyChecking=no \
                            target/*.jar \
                            ${APP_USER}@${APP_SERVER}:${APP_DIR}/app.jar

                        echo "======================================"
                        echo "JAR copied successfully"
                        echo "======================================"

                        ssh -o BatchMode=yes \
                            -o StrictHostKeyChecking=no \
                            ${APP_USER}@${APP_SERVER} \
                            "ls -lh ${APP_DIR}/app.jar"

                        echo "======================================"
                        echo "Restarting Spring Boot application"
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
            echo 'BUILD AND DEPLOYMENT SUCCESSFUL'
        }

        failure {
            echo 'BUILD OR DEPLOYMENT FAILED'
        }
    }
}
