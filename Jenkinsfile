
pipeline {
    agent any

    environment {
        EC2_HOST = '23.20.38.120'
        EC2_USER = 'ubuntu'
        DEPLOY_PATH = '/var/www/html'
        SSH_CREDENTIALS = 'ec2-website-ssh'
        KNOWN_HOSTS = '/var/lib/jenkins/.ssh/known_hosts'
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

                sshagent(credentials: [env.SSH_CREDENTIALS]) {
                    sh '''
                        set -eu

                        # SSH options
                        SSH_OPTIONS="-o BatchMode=yes -o StrictHostKeyChecking=yes -o UserKnownHostsFile=${KNOWN_HOSTS}"

                        # Verify SSH connection and destination directory
                        ssh $SSH_OPTIONS \
                            ${EC2_USER}@${EC2_HOST} \
                            "test -d ${DEPLOY_PATH}"

                        # Copy website file to the website EC2
                        scp $SSH_OPTIONS \
                            index.html \
                            ${EC2_USER}@${EC2_HOST}:/tmp/index.html

                        # Install the file and validate Nginx configuration
                        ssh $SSH_OPTIONS \
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
