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
                    def ec2Host = '18.221.132.15'
                    def ec2User = 'ubuntu'
                    def sshKeyId = 'dev-sshkey'

                   withCredentials([sshUserPrivateKey(credentialsId: 'dev-sshkey', keyFileVariable:'SSH_KEY')]) {
                        sh """
                        echo "Copying code to EC2 instance for"
                        scp -i $SSH_KEY -o StrictHostKeyChecking=no -r . ${ec2User}@${ec2Host}:${APP_DIR}

                        echo "Connecting to EC2 and setting up the app"
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no ${ec2User}@${ec2Host} << EOF
                            cd ${APP_DIR}
			    sudo apt install python3.12-venv
                            python3 -m venv venv || true
                            source venv/bin/activate
                            pip install -r requirements.txt
                            pkill -f "flask run" || true
                            nohup flask --app=admin.py run --host=0.0.0.0 --port=5000 &
                        EOF
                        """
                    }
                }
            }
        }
    }
}

