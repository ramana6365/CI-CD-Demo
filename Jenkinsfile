pipeline {
    agent any

    environment {
        EC2_IP = "13.55.24.135"   // Replace with your EC2 public IP
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building Node.js app...'
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                echo 'Running basic tests...'
                sh 'node -v'
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['my-ec2-key']) {
                    sh """
                    echo "Deploying Node app to EC2..."
                    ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} '
                        sudo apt update -y &&
                        sudo apt install -y nodejs npm &&
                        sudo mkdir -p /home/ubuntu/nodeapp &&
                        cd /home/ubuntu/nodeapp &&
                        sudo rm -rf * &&
                        exit
                    '

                    scp -o StrictHostKeyChecking=no -r * ubuntu@${EC2_IP}:/home/ubuntu/nodeapp/

                    ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} '
                        cd /home/ubuntu/nodeapp &&
                        npm install &&
                        nohup node app.js > output.log 2>&1 &
                    '
                    """
                }
            }
        }
    }
}
stage('Rollback') {
    steps {
        echo "Rolling back to previous stable version..."
        sh '''
        ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/mykey.pem ubuntu@3.27.181.246 '
        cd /home/ubuntu/app && git reset --hard HEAD~1 && sudo systemctl restart <your-app-service>'
        '''
    }
}
