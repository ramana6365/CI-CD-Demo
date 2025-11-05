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
        sshagent(['my-ec2-key']) {
          sh '''
            ssh -o StrictHostKeyChecking=no ubuntu@15.134.81.110 "
              cd /home/ubuntu/CI-CD-Demo &&
              git pull origin main &&
              sudo systemctl restart sample-app.service
            "
          '''
        }
      }
    }

    stage('AI Release Notes') {
      environment {
        OPENAI_API_KEY = credentials('OPENAI_API_KEY')
      }
      steps {
        sshagent(['github-deploy-key']) {
          sh '''
            set -e
            echo "Generating AI-powered release notes..."

            git log -3 --pretty=format:"%h - %s (%an)" > commits.txt
            COMMITS=$(cat commits.txt | jq -Rs .)

            cat > payload.json <<EOF
{
  "model": "gpt-4o-mini",
  "messages": [
    {"role": "system", "content": "You are a professional release note writer."},
    {"role": "user", "content": "Write concise, human-readable release notes for these commits: ${COMMITS}"}
  ]
}
EOF

            echo "Calling OpenAI API..."
            API_RESPONSE=$(curl -s https://api.openai.com/v1/chat/completions \
              -H "Content-Type: application/json" \
              -H "Authorization: Bearer $OPENAI_API_KEY" \
              -d @payload.json)

            echo "$API_RESPONSE" | jq . > api_raw.json
            AI_NOTES=$(echo "$API_RESPONSE" | jq -r '.choices[0].message.content // .error.message')

            echo "## AI Release Notes - $(date)" > release_notes.md
            echo "$AI_NOTES" >> release_notes.md
            cat release_notes.md

            git config --global --add safe.directory /var/lib/jenkins/workspace/ci_cd
            git config user.email "ramana@ci.local"
            git config user.name "Jenkins CI"

            git fetch origin main
            git checkout main || git checkout -b main origin/main
            git add release_notes.md
            git commit -m "docs: add AI-generated release notes [ci skip]" || true
            git pull origin main --rebase || git stash && git pull origin main --rebase && git stash pop || true
            git push origin main
          '''
        }
      }
    }

    stage('Rollback') {
      steps {
        echo "Rolling back to previous stable version..."
        sshagent(['my-ec2-key']) {
          sh '''
            ssh -o StrictHostKeyChecking=no ubuntu@15.134.81.110 "
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
    success {
      echo "Deployment pipeline completed successfully."
    }
    failure {
      echo "Build or deployment failed."
    }
  }
}
