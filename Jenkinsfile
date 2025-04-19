pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/Rox1999/star-agile-insurance-project.git'
        TEST_SERVER = '3.110.174.104'
        PROD_SERVER = '43.204.212.193'
        REMOTE_USER = 'root'
        APP_NAME = 'insurance-app'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: "${REPO_URL}"
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${APP_NAME}:latest .
                    docker save ${APP_NAME}:latest -o ${APP_NAME}.tar
                """
            }
        }

        stage('Deploy to Test Server') {
            steps {
                sshagent (credentials: ['root-server-key']) {
                    sh """
                        scp ${APP_NAME}.tar ${REMOTE_USER}@${TEST_SERVER}:/root/
                        ssh ${REMOTE_USER}@${TEST_SERVER} '
                            docker load -i /root/${APP_NAME}.tar &&
                            docker stop ${APP_NAME} || true &&
                            docker rm ${APP_NAME} || true &&
                            docker run -d --name ${APP_NAME} -p 8080:8080 ${APP_NAME}:latest
                        '
                    """
                }
            }
        }

        stage('Approval for Prod') {
            steps {
                input message: 'Proceed to Production Deployment?'
            }
        }

        stage('Deploy to Prod Server') {
            steps {
                sshagent (credentials: ['root-server-key']) {
                    sh """
                        scp ${APP_NAME}.tar ${REMOTE_USER}@${PROD_SERVER}:/root/
                        ssh ${REMOTE_USER}@${PROD_SERVER} '
                            docker load -i /root/${APP_NAME}.tar &&
                            docker stop ${APP_NAME} || true &&
                            docker rm ${APP_NAME} || true &&
                            docker run -d --name ${APP_NAME} -p 8080:8080 ${APP_NAME}:latest
                        '
                    """
                }
            }
        }
    }
}
