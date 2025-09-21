pipeline {
    agent any

    parameters {
        // Fetch sites dynamically
        activeChoiceParam(
            name: 'SITE',
            description: 'Select the site',
            filterable: true,
            choiceType: 'SINGLE_SELECT',
            groovyScript: [
                sandbox: true,
                script: """
                    import groovy.json.JsonSlurper
                    def url = 'http://192.168.1.254:8000/api/dcim/sites/'
                    def token = '1479d3740f85e8ab5900b72d31b89cb81fdc2a06'
                    def connection = new URL(url).openConnection()
                    connection.setRequestProperty('Authorization', 'Token ' + token)
                    connection.setRequestProperty('Accept', 'application/json')
                    def response = connection.inputStream.text
                    def json = new JsonSlurper().parseText(response)
                    return json.results.collect { it.name }.sort()
                """
            ]
        )

        // Fetch nodes dynamically based on selected SITE
        activeChoiceReactiveParam(
            name: 'NODE',
            description: 'Select node or ALL',
            filterable: true,
            choiceType: 'SINGLE_SELECT',
            groovyScript: [
                sandbox: true,
                script: """
                    import groovy.json.JsonSlurper
                    def site = params.SITE
                    def url = 'http://192.168.1.254:8000/api/dcim/devices/?site=' + site
                    def token = '1479d3740f85e8ab5900b72d31b89cb81fdc2a06'
                    def connection = new URL(url).openConnection()
                    connection.setRequestProperty('Authorization', 'Token ' + token)
                    connection.setRequestProperty('Accept', 'application/json')
                    def response = connection.inputStream.text
                    def json = new JsonSlurper().parseText(response)
                    def hosts = json.results.collect { it.name }.sort()
                    return ['ALL'] + hosts
                """
            ]
        )
    }

    environment {
        NETBOX_TOKEN = '1479d3740f85e8ab5900b72d31b89cb81fdc2a06'
        NETBOX_API   = 'http://192.168.1.254:8000/api/'
        GITHUB_TOKEN = credentials('github-cred-token')
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
                export NETBOX_API="${NETBOX_API}"
                export NETBOX_TOKEN="${NETBOX_TOKEN}"

                if [ "${NODE}" = "ALL" ]; then
                    ansible-playbook -i netbox_inv.yml generate_config.yml
                else
                    ansible-playbook -i netbox_inv.yml generate_config.yml --limit ${NODE}
                fi
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
                    git commit -m "Automated update from Jenkins build ${BUILD_NUMBER}"
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
            echo 'Pipeline finished.'
        }
    }
}
