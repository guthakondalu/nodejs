pipeline {
  agent any

  options {
    timestamps()
    ansiColor('xterm')
    buildDiscarder(logRotator(numToKeepStr: '5', daysToKeepStr: '10'))
  }
  environment {
       HOME="$WORKSPACE"
    }
  
  stages {
        
    stage('Build') {
      steps {
          sh """
                    echo "Tool versions"
                    node --version
                    npm --version
                    
                    echo HOME: \$HOME
                    npm install
                    
                """      
      }
    }
     
    stage('Test') {
      steps {
        sh "npm test"
      }
    }
    
    stage('Commit Code Coverage Metrics to GIT') {
      steps {
        sh "git add -f coverage"
        sh "git commit -m 'Code coverage metrics added from buil' $BUILD_NUMBER"
      }
    }
  }

post {
        success {
               	cleanWs()
            }
        }
        failure {
            echo 'Triggering email'
           
        }
    }


}