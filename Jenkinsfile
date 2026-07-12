pipeline {
    agent any

    environment {
        VM_IP = '20.245.136.71'
        JAR_NAME = 'devops-demo-0.3.0.jar'
        APP_DIR = '/opt/devops-demo'
        SERVICE_NAME = 'devops-demo'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                    sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                    sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/TEST-*.xml'
                }
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Deploy') {
            steps {
                sshagent(credentials: ['azure-vm-ssh']) {
                    sh """
                        scp -o StrictHostKeyChecking=no target/${JAR_NAME} azureuser@${VM_IP}:${APP_DIR}/${JAR_NAME}
                        ssh -o StrictHostKeyChecking=no azureuser@${VM_IP} 'sudo systemctl restart ${SERVICE_NAME}'
                    """
                }
            }
        }

        stage('Health Check') {
            steps {
                sh "chmod +x scripts/health_check.sh && ./scripts/health_check.sh ${VM_IP}"
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed — check stage logs above.'
        }
    }
}
