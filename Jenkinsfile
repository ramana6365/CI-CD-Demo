pipeline {
  agent any

  environment {
    EC2_IP        = "15.134.81.110"
    APP_PATH      = "/home/ubuntu/CI-CD-Demo"
    SERVICE_NAME  = "sample-app.service"
    OPENAI_API_KEY = credentials('OPENAI_API_KEY')
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
        sshagent(['my-ec2-key']) {
          sh """
            ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} '
              cd ${APP_PATH} &&
              git pull origin main &&
              sudo systemctl restart ${SERVICE_NAME}
            '
          """
        }
      }
    }

    stage('AI Release Notes') {
      steps {
        sshagent(['github-deploy-key']) {
          sh '''
            set -e
            echo "Generating AI-powered release notes..."

            # Collect the last 3 commits
            git log -3 --pretty=format:"%h - %s (%an)" > commits.txt

            # Send commit info to OpenAI API
            AI_NOTES=$(curl -s https://api.openai.com/v1/chat/completions \
              -H "Content-Type: application/json" \
              -H "Authorization: Bearer $OPENAI_API_KEY" \
              -d '{
                "model": "gpt-4o-mini",
                "messages": [
                  {"role": "system", "content": "You are a professional release note writer."},
                  {"role": "user", "content": "Write clear, concise release notes for these commits:\\n'"$(cat commits.txt)"'"}
                ]
              }' | jq -r '.choices[0].message.content')

            echo "## AI Release Notes - $(date)" > release_notes.md
            echo "$AI_NOTES" >> release_notes.md
            cat release_notes.md

            git config --global --add safe.directory /var/lib/jenkins/workspace/ci_cd
            git config user.email "ramana@ci.local"
            git config user.name "Jenkins CI"

            git fetch origin main
            git checkout main || git checkout -b main origin/main
            git pull origin main --rebase
            git add release_notes.md
            git commit -m "docs: add AI-generated release notes [ci skip]" || true
            git push origin main
          '''
        }
      }
    }

    stage('Rollback') {
      steps {
        echo "Rolling back to previous version..."
        sshagent(['my-ec2-key']) {
          sh """
            ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} '
              cd ${APP_PATH} &&
              git reset --hard HEAD~1 &&
              sudo systemctl restart ${SERVICE_NAME}
            '
          """
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
