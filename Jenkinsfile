pipeline {
    agent any
    options {
        skipDefaultCheckout(true)   // Disable default HTTPS checkout
    }

    environment {
        EC2_IP        = "3.26.97.57"
        APP_PATH      = "/home/ubuntu/CI-CD-Demo"
        SERVICE_NAME  = "sample-app.service"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Checking out code via SSH..."
                sshagent(['github-ssh-key']) {       // your GitHub deploy key ID
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
                sshagent (credentials: ['my-ec2-key1']) {    // EC2 key
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
                sshagent(['github-ssh-key']) {
                    sh '''
                    set -e
                    mkdir -p ~/.ssh
                    ssh-keyscan -t rsa github.com >> ~/.ssh/known_hosts 2>/dev/null || true

                    echo "Generating AI release notes..."
                    echo "Release Notes - $(date)" > release_notes.txt
                    echo "Changes deployed from latest Git commit:" >> release_notes.txt
                    git log -1 --pretty=format:"%h - %s (%an)" >> release_notes.txt
                    echo "AI Release Notes generated." >> release_notes.txt
                    cat release_notes.txt

                    git config --global --add safe.directory /var/lib/jenkins/workspace/ci_cd
                    git config user.email "ramana@ci.local"
                    git config user.name "Jenkins CI"

                    # Ensure we are on main
                    git fetch origin main
                    git checkout main || git checkout -b main origin/main
                    git pull origin main --rebase

                    git add release_notes.txt
                    if ! git diff --cached --quiet; then
                        git commit -m "chore: add AI-generated release notes [ci skip]" || true
                        echo "Pushing release notes to GitHub via SSH..."
                        git push origin main
                    else
                        echo "No changes to commit. Skipping push."
                    fi
                    '''
                }
            }
        }

        stage('Rollback') {
            steps {
                echo "Rolling back to previous stable version..."
                sshagent (credentials: ['my-ec2-key1']) {
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
