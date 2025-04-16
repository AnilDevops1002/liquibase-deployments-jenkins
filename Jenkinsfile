pipeline {
    agent any

    environment {
        EC2_HOST = credentials('postgres-host')
        EC2_USER = 'ubuntu'
        EC2_SSH_KEY = credentials('ec2-db-ssh-key')
        POSTGRES_USER = credentials('postgres-user')
        POSTGRES_PASSWORD = credentials('postgres-password')
        EMAIL_APP_PASSCODE = credentials('email-app-passcode')
    }

    parameters {
        string(name: 'POSTGRES_DB', defaultValue: 'paycio', description: 'Database to deploy changes to')
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Tag of Docker image')
    }

    stages {
        stage('Deploy to EC2') {
            steps {
                sshagent (credentials: ['ec2-db-ssh-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no $EC2_USER@$EC2_HOST << EOF
                        echo "✅ Logged into EC2 instance"

                        IMAGE="ghcr.io/hareeshkhs/liquibase-schema-manager:${IMAGE_TAG}"

                        echo "📦 Pulling Docker image: \$IMAGE"
                        docker pull \$IMAGE

                        echo "🧹 Cleaning old containers"
                        docker rm -f liquibase-schema-manager || true

                        echo "🚀 Running new container"
                        docker run --name liquibase-schema-manager -d \
                          -e POSTGRES_HOST=${EC2_HOST} \
                          -e POSTGRES_DB=${POSTGRES_DB} \
                          -e POSTGRES_USER=${POSTGRES_USER} \
                          -e POSTGRES_PASSWORD=${POSTGRES_PASSWORD} \
                          -e EMAIL_APP_PASSCODE=${EMAIL_APP_PASSCODE} \
                          \$IMAGE

                        echo "✅ Deployment finished"
                    EOF
                    """
                }
            }
        }
    }
}
