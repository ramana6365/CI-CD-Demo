pipeline {
    agent any

    environment {
        EC2_IP = "3.26.97.57"
        APP_PATH = "/home/ubuntu/CI-CD-Demo"
        SERVICE_NAME = "sample-app.service"
    }

    stages {

        stage('Checkout') {
            steps {
                sshagent(['my-ec2-key']) {
                    git branch: 'main', url: 'git@github.com:ramana6365/CI-CD-Demo.git'
                }
            }
        }

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
                sshagent(['my-ec2-key']) {
                    sh '''
                    set -e
                    mkdir -p ~/.ssh
                    ssh-keyscan -t rsa github.com >> ~/.ssh/known_hosts 2>/dev/null || true

                    echo "Generating AI release notes..."
                    echo "## Release Notes - $(date)" > release_notes.md
                    echo "Changes deployed from latest Git commit:" >> release_notes.md
                    git log -1 --pretty=format:"- %h - %s (%an)" >> release_notes.md
                    echo "AI Release Notes generated." >> release_notes.md
                    cat release_notes.md

                    # Configure git
                    git config --global --add safe.directory /var/lib/jenkins/workspace/ci_cd
                    git config user.email "ramana@ci.local"
                    git config user.name "Jenkins CI"

                    # Checkout main branch before commit
                    git fetch origin main
                    git checkout main || git checkout -b main origin/main
                    git pull origin main --rebase

                    # Commit and push release notes
                    git add release_notes.md
                    if ! git diff --cached --quiet; then
                        git commit -m "docs: add AI-generated release notes [ci skip]" || true
                        echo "Pushing release notes to GitHub via SSH..."
                        git push origin main
                    else
                        echo "No changes to commit, skipping push."
                    fi
                    '''
                }
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
