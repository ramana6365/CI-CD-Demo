pipeline {
  agent any

  environment {
    EC2_IP       = "15.134.81.110"
    APP_PATH     = "/home/ubuntu/CI-CD-Demo"
    SERVICE_NAME = "sample-app.service"
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

            git log -3 --pretty=format:"%h - %s (%an)" > commits.txt
            COMMITS=$(cat commits.txt | jq -Rs .)

            jq -n --arg commits "$COMMITS" '{
              model: "gpt-4o-mini",
              messages: [
                {role: "system", content: "You are a professional release note writer."},
                {role: "user", content: ("Write concise, human-readable release notes for these commits: " + $commits)}
              ]
            }' > payload.json

            echo "Calling OpenAI API..."
            curl -s https://api.openai.com/v1/chat/completions \
              -H "Content-Type: application/json" \
              -H "Authorization: Bearer $OPENAI_KEY" \
              -d @payload.json > api_raw.json

            jq . api_raw.json > api_raw_pretty.json || true
            AI_NOTES=$(jq -r '.choices[0].message.content // .error.message' api_raw.json)

            echo "## AI Release Notes - $(date)" > release_notes.md
            echo "$AI_NOTES" >> release_notes.md
            cat release_notes.md

            git config --global --add safe.directory /var/lib/jenkins/workspace/ci_cd
            git config user.email "ramana@ci.local"
            git config user.name "Jenkins CI"

            git fetch origin main
            git checkout main
            git stash || true
            git pull origin main --rebase
            git stash pop || true

            git add release_notes.md api_raw.json api_raw_pretty.json
            git commit -m "docs: add AI-generated release notes + raw response [ci skip]" || true
            git push origin main
          '''
        }
      }
    }

    stage('Rollback Suggestion') {
      when {
        expression { currentBuild.currentResult == 'FAILURE' || currentBuild.currentResult == 'UNSTABLE' }
      }
      steps {
        script {
          echo "Build failed or unstable — generating rollback suggestion..."
          def lastGoodBuild = currentBuild.rawBuild.getPreviousSuccessfulBuild()
          if (lastGoodBuild) {
            echo "Last successful build: #${lastGoodBuild.number}"

            sh '''
              set -e
              LAST_GOOD_COMMIT=$(git rev-parse HEAD~1 || true)
              echo "## Rollback Suggestion - $(date)" > rollback_suggestion.md
              echo "The current build ($BUILD_NUMBER) failed or is unstable." >> rollback_suggestion.md
              echo "Suggested rollback target: Build #${lastGoodBuild.number}" >> rollback_suggestion.md
              echo "Commit ID: $LAST_GOOD_COMMIT" >> rollback_suggestion.md
              echo "" >> rollback_suggestion.md
              echo "To rollback, run:" >> rollback_suggestion.md
              echo "\\n  git checkout $LAST_GOOD_COMMIT" >> rollback_suggestion.md
              echo "\\n  git push origin main --force" >> rollback_suggestion.md
              cat rollback_suggestion.md

              git config --global --add safe.directory /var/lib/jenkins/workspace/ci_cd
              git config user.email "ramana@ci.local"
              git config user.name "Jenkins CI"

              git add rollback_suggestion.md
              git commit -m "docs: add rollback suggestion for failed build [ci skip]" || true
              git push origin main || true
            '''
          } else {
            echo "No previous successful build found — cannot generate rollback suggestion."
          }
        }
      }
    }
  }  // closes stages

  post {
    success { echo "Deployment pipeline completed successfully." }
    failure { echo "Build or deployment failed." }
  }
}  
