pipeline {
    agent any

    environment {
        // Git repo and branch info
        GIT_REPO = 'git@github.com:bhumireddysurekha7/pythonflask.git'

        // Environment-specific hosts and deployment paths
        DEV_HOST = '3.140.217.63'
        STAGE_HOST = '3.16.169.103'
        PROD_HOST = '18.118.14.180'

        EC2_USER = 'ubuntu'  // Change to your VM user
        APP_DIR = '/home/ubuntu/flask-app'
    }
        stage('Deploy') {
            steps {
                sshagent (credentials: ["${env.SSH_CREDENTIALS_ID}"]) {
                    script {
                        def deployHost = ''
                        
                        // Select target host based on branch
                        if (env.BRANCH_NAME == 'dev') {
                            deployHost = env.DEV_HOST
                            SSH_CREDENTIALS_ID = 'dev-sshkey'
                        } else if (env.BRANCH_NAME == 'stage') {
                            deployHost = env.STAGE_HOST
                            SSH_CREDENTIALS_ID = 'stage-sshkey'
                        } else if (env.BRANCH_NAME == 'main') {
                            deployHost = env.PROD_HOST
                            SSH_CREDENTIALS_ID = 'prod-sshkey'
                        } else {
                            error "Unsupported branch: ${env.BRANCH_NAME}"
                        }

                        // Deploy script to the selected environment
                        sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${deployHost} << EOF
                            echo "Deploying to ${deployHost} from branch ${env.BRANCH_NAME}"
                            rm -rf ${APP_DIR}
                            git clone ${GIT_REPO} ${APP_DIR}
                            cd ${APP_DIR}
                            git checkout ${env.BRANCH_NAME}

                            # Setup virtual environment and dependencies
                            python3 -m venv venv
                            source venv/bin/activate
                            pip install --upgrade pip
                            pip install -r requirements.txt

                            # Run Flask (replace with gunicorn or systemctl for prod)
                            nohup flask run --host=0.0.0.0 --port=5000 > flask.log 2>&1 &
                        EOF
                        """
                    }
                }
            }
        }
    }

    post {
        failure {
            echo 'Deployment failed!'
        }
        success {
            echo 'Deployment completed successfully.'
        }
    }
}


