pipeline {
    agent any

    environment {
        EC2_IP = "3.26.97.57"
        APP_PATH = "/home/ubuntu/CI-CD-Demo"
        SERVICE_NAME = "sample-app.service"
        GITHUB_TOKEN = "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"
    }

    stages {
        stage('Build') {
            steps {
                echo "Building the Node.js app..."
                sh '''
                npm install
                echo "Build completed successfully."
                '''
            }
        }

        stage('Deploy to EC2') {
            steps {
                echo "Deploying to EC2 (${EC2_IP})..."
                sshagent (credentials: ['my-ec2-key']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@3.26.97.57 "
                        cd /home/ubuntu/CI-CD-Demo &&
                        git pull origin main &&
                        sudo systemctl restart sample-app.service
                    "
                    '''
                }
            }
        }

       stage('AI Release Notes') {
    steps {
        echo "Generating AI release notes..."
        sh '''
        # Generate release notes file
        echo "Release Notes - $(date)" > release_notes.txt
        echo "Changes deployed from latest Git commit:" >> release_notes.txt
        git log -1 --pretty=format:"%h - %s (%an)" >> release_notes.txt
        echo "AI Release Notes generated." >> release_notes.txt
        cat release_notes.txt

        # Git safe directory
        git config --global --add safe.directory /var/lib/jenkins/workspace/ci_cd

        # Configure user info
        git config user.email "ramana@ci.local"
        git config user.name "Jenkins CI"

        # Fix the remote URL (no double repo path)
        git remote set-url origin https://ramana6365:${GITHUB_TOKEN}@github.com/ramana6365/CI-CD-Demo.git

        # Create or checkout main safely
        git fetch origin main
        git checkout main || git checkout -b main origin/main

        # Commit and push changes if new content exists
        git add release_notes.txt
        if ! git diff --cached --quiet; then
          git commit -m "chore: add AI-generated release notes [ci skip]" || echo "No new changes to commit."
          git pull origin main --rebase
          git push origin main
        else
          echo "No new release notes to commit."
        fi
        '''
    }
}




        stage('Rollback') {
            steps {
                echo "Rolling back to previous stable version..."
                sshagent (credentials: ['my-ec2-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} '
                        cd ${APP_PATH} &&
                        git reset --hard HEAD~1 &&
                        sudo systemctl restart ${SERVICE_NAME}'
                    """
                }
                echo "Rollback completed successfully."
            }
        }
    }

    post {
        success {
            echo "Deployment pipeline completed successfully!"
        }
        failure {
            echo "Build or deployment failed."
        }
    }
}
