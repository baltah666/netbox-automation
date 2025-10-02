pipeline {
    agent any

    environment {
        VENV_PATH = "${WORKSPACE}/venv"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/baltah666/netbox-automation.git', credentialsId: 'netbox-generate-configFile'
            }
        }

        stage('Setup Environment') {
            steps {
                sh """
                python3 -m venv ${VENV_PATH}
                . ${VENV_PATH}/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                ansible-galaxy collection install netbox.netbox --force
                """
            }
        }

        stage('Run Automation Scripts in Parallel') {
            steps {
                script {
                    // 1. Get all sites from NetBox dynamically
                    def sites = sh(
                        script: """
                        . ${VENV_PATH}/bin/activate
                        python3 - <<EOF
import pynetbox
import os

NETBOX_URL = os.getenv('NETBOX_URL', 'http://netbox.example.com')
NETBOX_TOKEN = os.getenv('NETBOX_TOKEN', 'your_token_here')

nb = pynetbox.api(url=NETBOX_URL, token=NETBOX_TOKEN)
sites = [site.slug for site in nb.dcim.sites.all() if len(nb.dcim.devices.filter(site_id=site.id)) > 0]
print(",".join(sites))
EOF
                        """,
                        returnStdout: true
                    ).trim().split(',')

                    echo "Sites with devices: ${sites}"

                    // 2. Build parallel stages
                    def parallelStages = [:]
                    for (site in sites) {
                        parallelStages["Site: ${site}"] = {
                            sh """
                            . ${VENV_PATH}/bin/activate
                            ansible-playbook -i netbox_inv.yml generate_config.yml --limit sites_${site}
                            """
                        }
                    }

                    // 3. Execute in parallel
                    parallel parallelStages
                }
            }
        }

        stage('Git Push Changes') {
            steps {
                sh """
                git add .
                git commit -m "Updated device configs"
                git push origin main
                """
            }
        }
    }

    post {
        always {
            echo "Pipeline finished"
        }
        failure {
            echo "Pipeline failed"
        }
    }
}
