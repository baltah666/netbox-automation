pipeline {
    agent any
    environment {
        VIRTUAL_ENV = "${WORKSPACE}/venv"
        PATH = "${WORKSPACE}/venv/bin:${env.PATH}"
        NETBOX_API = "http://192.168.1.254:8000/"
        NETBOX_TOKEN = "1479d3740f85e8ab5900b72d31b89cb81fdc2a06"
    }
    stages {
        stage('Setup Environment') {
            steps {
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                ansible-galaxy collection install netbox.netbox --force
                '''
            }
        }

        stage('Run Automation Scripts in Parallel') {
            steps {
                script {
                    // Define your sites dynamically
                    def sites = ['lab_test_env', 'another_env']  // You can also fetch this from NetBox API
                    def parallelStages = [:]

                    for (site in sites) {
                        // Map to inventory group
                        def inventoryGroup = "sites_${site}"
                        parallelStages["Site: ${site}"] = {
                            sh """
                            . venv/bin/activate
                            ansible-playbook -i netbox_inv.yml generate_config.yml --limit ${inventoryGroup}
                            """
                        }
                    }

                    parallel parallelStages
                }
            }
        }

        stage('Git Push Changes') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                sh '''
                git add .
                git commit -m "Updated configs from Jenkins pipeline"
                git push origin main
                '''
            }
        }
    }

    post {
        always {
            echo "Pipeline finished."
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
