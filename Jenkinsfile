pipeline {
  agent any

  environment {
    DEPLOY_DIR = "/opt/sample-app"
    DEPLOY_LOG = "/tmp/deploy.log"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      steps {
        sh '''
          echo "Installing Node.js if not installed..."
          if ! command -v node >/dev/null 2>&1; then
            curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
            sudo apt-get install -y nodejs
          fi
          echo "Node.js version:"
          node -v
        '''
      }
    }

    stage('Deploy') {
      steps {
        sh '''
          echo "Deploying application..." | tee ${DEPLOY_LOG}
          sudo mkdir -p ${DEPLOY_DIR}
          sudo cp -r * ${DEPLOY_DIR}/
          cd ${DEPLOY_DIR}
          nohup npm start > app.log 2>&1 &
          echo "Application started on port 3000"
        '''
      }
    }
  }
}
