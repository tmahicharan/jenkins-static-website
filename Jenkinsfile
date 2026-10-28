pipeline {
    agent any

    environment {
        EC2_HOST = '23.20.38.120'
        EC2_USER = 'ubuntu'
        DEPLOY_PATH = '/var/www/html'
        SSH_CREDENTIALS = 'ec2-website-ssh'
    }

    stages {

        stage('Code Checkout') {
            steps {
                echo 'Checking out source code from GitHub'

                git branch: 'main',
                    url: 'https://github.com/tmahicharan/jenkins-static-website.git'
            }
        }

        stage('Validation') {
            steps {
                echo 'Validating website files'

                sh '''
                    set -eu

                    test -s index.html
                    grep -qi '<html' index.html

                    echo "Validation successful"
                '''
            }
        }

        stage('Deployment') {
            steps {
                echo 'Deploying website to AWS EC2'

                sshagent(credentials: ['ec2-website-ssh']) {
                    sh '''
                        set -eu

                        # Verify SSH connectivity
                        ssh -o BatchMode=yes \
                            -o StrictHostKeyChecking=yes \
                            ${EC2_USER}@${EC2_HOST} \
                            "test -d ${DEPLOY_PATH}"

                        # Copy website file to EC2
                        scp -o BatchMode=yes \
                            -o StrictHostKeyChecking=yes \
                            index.html \
                            ${EC2_USER}@${EC2_HOST}:/tmp/index.html

                        # Publish the website using sudo
                        ssh -o BatchMode=yes \
                            -o StrictHostKeyChecking=yes \
                            ${EC2_USER}@${EC2_HOST} \
                            "sudo install -m 644 /tmp/index.html ${DEPLOY_PATH}/index.html && rm /tmp/index.html && sudo nginx -t"

                        echo "Deployment successful"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully'
        }

        failure {
            echo 'Pipeline failed. Check the Jenkins console output'
        }
    }
}
