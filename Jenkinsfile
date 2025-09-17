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
                sh '''#!/bin/bash
                # Create virtual environment if not exists
                if [ ! -d venv ]; then
                    python3 -m venv venv
                fi

                # Activate venv and install dependencies
                . venv/bin/activate
                pip install --upgrade pip
                if [ -f requirements.txt ]; then
                    pip install -r requirements.txt
                fi

                # Install NetBox collection (idempotent)
                ansible-galaxy collection install netbox.netbox || true
                '''
            }
        }

        stage('Run Automation Script') {
            steps {
                sh '''#!/bin/bash
                . venv/bin/activate
                ansible-playbook -i netbox_inv.yml generate_config.yml
                '''
            }
        }

        stage('Push Generated Configs to GitHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-cred', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    sh '''#!/bin/bash
                    # Configure Git user
                    git config user.name "Abdulilah Baltah"
                    git config user.email "baltah666@gmail.com"

                    # Set remote URL with token authentication
                    git remote set-url origin https://github.com/baltah666/netbox-automation.git

                    # Add all generated files
                    git add .

                    # Commit only if there are changes
                    if ! git diff --cached --quiet; then
                        git commit -m "Automated config update by Jenkins"
                        git push origin main
                    else
                        echo "No changes to commit."
                    fi
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
    }
}
