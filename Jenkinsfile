
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Prepare Python Environment') {
            steps {
                sh '''
                    set -eu


                    sudo mkdir -p /opt/python-app
                    sudo rsync -a \
                        --exclude='.git/' \
                        --exclude='venv/' \
                        --exclude='.venv/' \
                        ./ /opt/python-app/

                    sudo chown -R appuser:appuser /opt/python-app

                    sudo -u appuser python3 -m venv /opt/python-app/venv

                    sudo -u appuser /opt/python-app/venv/bin/pip \
                        install -r /opt/python-app/requirements.txt

                    sudo -u appuser /opt/python-app/venv/bin/pip \
                        install gunicorn
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    set -eu

                    sudo systemctl restart flaskapp.service
                    sleep 2

                    sudo systemctl is-active --quiet flaskapp.service

                    echo "Deployment completed successfully"
                '''
            }
        }
    }
}
