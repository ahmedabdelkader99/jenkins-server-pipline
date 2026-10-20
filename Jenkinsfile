pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        PROD_HOST = '45.220.54.15'
        PROD_USER = 'appuser'
        APP_DIR = '/opt/python-app'
        SSH_KEY = '/var/lib/jenkins/.ssh/production_deploy'
        SERVICE_NAME = 'flaskapp.service'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Project') {
            steps {
                sh '''
                    set -eu

                    test -f app.py
                    test -f requirements.txt

                    echo "Project files validated successfully"
                '''
            }
        }

        stage('Test SSH Connection') {
            steps {
                sh '''
                    set -eu

                    chmod 600 "$SSH_KEY"

                    ssh -i "$SSH_KEY" \
                        -o BatchMode=yes \
                        -o ConnectTimeout=10 \
                        "$PROD_USER@$PROD_HOST" \
                        'echo "SSH connection successful"'
                '''
            }
        }

        stage('Prepare Production') {
            steps {
                sh '''
                    set -eu

                    ssh -i "$SSH_KEY" \
                        -o BatchMode=yes \
                        "$PROD_USER@$PROD_HOST" \
                        'test -d /opt/python-app/venv &&
                         test -w /opt/python-app'
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    set -eu

                    rsync -az --delete \
                        --exclude='.git/' \
                        --exclude='venv/' \
                        --exclude='.venv/' \
                        --exclude='__pycache__/' \
                        --exclude='.env' \
                        --exclude='.pytest_cache/' \
                        -e "ssh -i $SSH_KEY -o BatchMode=yes" \
                        ./ "$PROD_USER@$PROD_HOST:$APP_DIR/"

                    ssh -i "$SSH_KEY" \
                        -o BatchMode=yes \
                        "$PROD_USER@$PROD_HOST" \
                        'set -eu
                         cd /opt/python-app
                         ./venv/bin/pip install -r requirements.txt
                         ./venv/bin/pip install gunicorn'
                '''
            }
        }

        stage('Restart Service') {
            steps {
                sh '''
                    set -eu

                    ssh -i "$SSH_KEY" \
                        -o BatchMode=yes \
                        "$PROD_USER@$PROD_HOST" \
                        'sudo -n systemctl restart flaskapp.service &&
                         sleep 2 &&
                         sudo -n systemctl is-active --quiet flaskapp.service'

                    echo "Deployment completed successfully"
                '''
            }
        }
    }

    post {
        success {
            echo 'Production deployment succeeded.'
        }

        failure {
            echo 'Deployment failed. Check the stage logs before retrying.'
        }
    }
}
