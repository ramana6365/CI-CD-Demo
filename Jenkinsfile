pipeline {
  agent any

  environment {
    EC2_IP        = "15.134.81.110"
    APP_PATH      = "/home/ubuntu/CI-CD-Demo"
    SERVICE_NAME  = "sample-app.service"
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
        sshagent(['my-ec2-key1']) {
          sh '''
            ssh -o StrictHostKeyChecking=no ubuntu@15.134.81.110"
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
        sshagent(['github-deploy-key']) {
          sh '''
            set -e
            echo "Generating AI release notes..."
            echo "## Release Notes - $(date)" > release_notes.md
            echo "Changes deployed from latest Git commit:" >> release_notes.md
            git log -1 --pretty=format:"- %h - %s (%an)" >> release_notes.md
            echo "AI Release Notes generated." >> release_notes.md
            cat release_notes.md

            git config --global --add safe.directory /var/lib/jenkins/workspace/ci_cd
            git config user.email "ramana@ci.local"
            git config user.name "Jenkins CI"

            git add release_notes.md
            git commit -m "docs: add AI-generated release notes [ci skip]" || true
            git pull origin main --rebase
            git push origin main
          '''
        }
      }
    }

    stage('Rollback') {
      steps {
        echo "Rolling back to previous stable version..."
        sshagent(['my-ec2-key1']) {
          sh '''
            ssh -o StrictHostKeyChecking=no ubuntu@15.134.81.110"
              cd /home/ubuntu/CI-CD-Demo &&
              git reset --hard HEAD~1 &&
              sudo systemctl restart sample-app.service
            "
          '''
        }
        echo "Rollback completed successfully."
      }
    }
  }

  post {
    success { echo "Deployment pipeline completed successfully!" }
    failure { echo "Build or deployment failed." }
  }
}
