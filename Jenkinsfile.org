pipeline {
    agent any

    environment {
        NETBOX_TOKEN = '1479d3740f85e8ab5900b72d31b89cb81fdc2a06'
        NETBOX_API   = 'http://192.168.1.254:8000/api/'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/baltah666/netbox-automation.git',
                    credentialsId: 'github-cred'
            }
        }

        stage('Setup Environment') {
            steps {
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install --upgrade pip
                if [ -f requirements.txt ]; then
                    pip install -r requirements.txt
                fi
                ansible-galaxy collection install netbox.netbox || true
                '''
            }
        }

        stage('Run Automation Script') {
            steps {
                sh '''
                . venv/bin/activate
                export NETBOX_API="http://192.168.1.254:8000/"
                export NETBOX_TOKEN="1479d3740f85e8ab5900b72d31b89cb81fdc2a06"
                ansible-playbook -i netbox_inv.yml generate_config.yml
                '''
            }
        }

    }

    post {
        always {
            echo 'Pipeline finished.'
        }
    }
}
