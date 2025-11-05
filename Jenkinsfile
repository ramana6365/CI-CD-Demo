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

    
  stage('AI Release Notes') {
  environment {
    OPENAI_KEY = credentials('OPENAI_API_KEY')
  }
  steps {
    sshagent(['github-deploy-key']) {
      sh '''
        set -e
        echo "Generating AI-powered release notes..."

        # Collect latest 3 commits
        git log -3 --pretty=format:"%h - %s (%an)" > commits.txt
        COMMITS=$(cat commits.txt | jq -Rs .)

        # Build proper JSON payload (fixed jq syntax)
        jq -n --arg commits "$COMMITS" '{
          model: "gpt-4o-mini",
          messages: [
            {role: "system", content: "You are a professional release note writer."},
            {role: "user", content: ("Write concise, human-readable release notes for these commits: \($commits)")}
          ]
        }' > payload.json

        echo "Calling OpenAI API..."
        curl -s https://api.openai.com/v1/chat/completions \
          -H "Content-Type: application/json" \
          -H "Authorization: Bearer $OPENAI_KEY" \
          -d @payload.json > api_raw.json

        # Pretty print JSON for readability
        jq . api_raw.json > api_raw_pretty.json || true

        # Extract the actual text content
        AI_NOTES=$(jq -r '.choices[0].message.content // .error.message' api_raw.json)

        # Write markdown release notes
        echo "## AI Release Notes - $(date)" > release_notes.md
        echo "$AI_NOTES" >> release_notes.md
        cat release_notes.md

        # Configure Git user
        git config --global --add safe.directory /var/lib/jenkins/workspace/ci_cd
        git config user.email "ramana@ci.local"
        git config user.name "Jenkins CI"

        # Ensure we are on main branch
        git fetch origin main
        git checkout main
        git pull origin main --rebase

        # Commit both AI release notes and raw responses
        git add release_notes.md api_raw.json api_raw_pretty.json
        git commit -m "docs: add AI-generated release notes + raw response [ci skip]" || true
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
