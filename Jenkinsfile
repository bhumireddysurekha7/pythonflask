pipeline {
    agent any

    environment {
        APP_DIR = '/home/ubuntu/flask-app'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'dev', url: 'https://github.com/bhumireddysurekha7/pythonflask.git'
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    def ec2Host = ''
                    def ec2User = 'ubuntu'
                    def sshKeyId = ''

                    if (env.BRANCH_NAME == 'dev') {
                        ec2Host = '18.221.132.15'
                        sshKeyId = 'dev-sshkey'
                    } else if (env.BRANCH_NAME == 'stage') {
                        ec2Host = '3.14.150.131'
                        sshKeyId = 'stage-sshkey'
                    } else if (env.BRANCH_NAME == 'main') {
                        ec2Host = '18.222.129.198'
                        sshKeyId = 'prod-sshkey'
                    } else {
                        error("Unsupported branch for deployment: ${env.BRANCH_NAME}")
                    }

                    sshagent (credentials: [sshKeyId]) {
                        sh """
                        echo "Copying code to EC2 instance for ${env.BRANCH_NAME}"
                        scp -o StrictHostKeyChecking=no -r . ${ec2User}@${ec2Host}:${APP_DIR}

                        echo "Connecting to EC2 and setting up the app"
                        ssh -o StrictHostKeyChecking=no ${ec2User}@${ec2Host} << EOF
                            cd ${APP_DIR}
                            python3 -m venv venv || true
                            source venv/bin/activate
                            pip install -r requirements.txt
                            pkill -f "flask run" || true
                            nohup flask run --host=0.0.0.0 --port=5000 &
                        EOF
                        """
                    }
                }
            }
        }
    }
}

