pipeline {
    agent any

    parameters {
        choice(
            name: 'SITE',
            choices: ['Lab-Test-Env', 'Solna', 'Uppsala'],
            description: 'Select the site'
        )
        string(
            name: 'DEVICES',
            defaultValue: 'All',
            description: 'Comma-separated list of devices (e.g., Test-Env-Access_sw01,Test-Env-DS_sw02) or All'
        )
    }

    environment {
        NETBOX_TOKEN = '1479d3740f85e8ab5900b72d31b89cb81fdc2a06'
        NETBOX_API   = 'http://192.168.1.254:8000/api/'
        GITHUB_TOKEN = credentials('github-cred-token') // <-- inject token
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

                # Prepare extra vars for Ansible
                EXTRA_VARS="site=${SITE}"
                if [ "${DEVICES}" != "All" ]; then
                    EXTRA_VARS="$EXTRA_VARS devices=${DEVICES}"
                fi

                ansible-playbook -i netbox_inv.yml generate_config.yml --extra-vars "$EXTRA_VARS"
                '''
            }
        }

        stage('Git Push Changes') {
            steps {
                sh '''
                git config user.name "Abdulilah Baltah"
                git config user.email "baltah666@gmail.com"

                if [ -n "$(git status --porcelain)" ]; then
                    git add .
                    git commit -m "Automated update from Jenkins build ${BUILD_NUMBER} - Site: ${SITE}, Devices: ${DEVICES}"
                    git push https://${GITHUB_TOKEN}@github.com/baltah666/netbox-automation.git main
                else
                    echo "No changes to commit."
                fi
                '''
            }
        }
    }

    post {
        always {
            echo "Pipeline finished for Site: ${SITE}, Devices: ${DEVICES}"
        }
    }
}
