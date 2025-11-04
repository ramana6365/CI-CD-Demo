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
        sh 'echo "Building the application..."'
      }
    }

    stage('Deploy') {
      steps {
        sh '''
          echo "Deploying application..." | tee ${DEPLOY_LOG}
          rsync -av --delete --exclude .git ./ ${DEPLOY_DIR}
          sudo systemctl restart sample-app || true
        '''
      }
    }
  }
}
