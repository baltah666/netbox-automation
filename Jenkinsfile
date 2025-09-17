pipeline {
    agent any

    environment {
        NETBOX_TOKEN = '1479d3740f85e8ab5900b72d31b89cb81fdc2a06'
        NETBOX_API = 'http://192.168.1.254:8000/api/'
    }

    stages {
        stage('Run in Docker') {
            steps {
                script {
                    docker.image('python:3.11-slim').inside('-v $WORKSPACE:/workspace') {
                        // Checkout code
                        checkout([$class: 'GitSCM',
                            branches: [[name: '*/main']],
                            userRemoteConfigs: [[url: 'https://github.com/baltah666/netbox-automation.git', credentialsId: 'github-cred']]
                        ])

                        // Setup environment
                        sh '''
                            python3 -m venv venv
                            . venv/bin/activate
                            pip install --upgrade pip
                            if [ -f requirements.txt ]; then
                                pip install -r requirements.txt
                            fi
                            ansible-galaxy collection install netbox.netbox || true
                        '''

                        // Run automation script
                        sh '''
                            . venv/bin/activate
                            ansible-playbook -i netbox_inv.yml generate_config.yml
                        '''

                        // Push changes to GitHub
                        withCredentials([usernamePassword(credentialsId: 'github-cred', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                            sh '''
                                git config user.name "Abdulilah Baltah"
                                git config user.email "baltah666@gmail.com"
                                git remote set-url origin https://$GIT_USER:$GIT_TOKEN@github.com/baltah666/netbox-automation.git
                                git add .
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
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
    }
}
