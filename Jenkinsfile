pipeline {
    agent any

    environment {
        EC2_IP = "3.26.97.57"
        APP_PATH = "/home/ubuntu/CI-CD-Demo"
        SERVICE_NAME = "sample-app.service"
    }

    stages {
        stage('Build') {
            steps {
                echo "Building the Node.js app..."
                sh '''
                npm install
                echo "✅ Build completed successfully."
                '''
            }
        }

        stage('Deploy to EC2') {
    steps {
        echo "Deploying to EC2 (3.26.97.57)..."
        sshagent (credentials: ['my-ec2-key']) {
            sh '''
            ssh -o StrictHostKeyChecking=no ubuntu@3.26.97.57 '
                cd /home/ubuntu/CI-CD-Demo &&
                git pull origin main &&
                sudo systemctl restart sample-app.service
            '
            '''
        }
    }
}

        stage('AI Release Notes') {
            steps {
                echo "🧠 Generating AI-based release notes..."
                sh '''
                echo "Release Notes - $(date)" > release_notes.txt
                echo "Changes deployed from latest Git commit:" >> release_notes.txt
                git log -1 --pretty=format:"%h - %s (%an)" >> release_notes.txt
                echo "✅ AI Release Notes generated."
                cat release_notes.txt
                '''
            }
        }

        stage('Rollback') {
            steps {
                echo "⏪ Rolling back to previous stable version..."
                sshagent (credentials: ['my-ec2-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} '
                    cd ${APP_PATH} &&
                    git reset --hard HEAD~1 &&
                    sudo systemctl restart ${SERVICE_NAME}'
                    """
                }
                echo "✅ Rollback completed successfully."
            }
        }
    }

    post {
        success {
            echo "🎉 Deployment pipeline completed successfully!"
        }
        failure {
            echo "❌ Build or deployment failed. Check logs for details."
        }
    }
}
