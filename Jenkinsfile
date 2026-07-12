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
