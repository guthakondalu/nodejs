pipeline {
  agent any
  
  stages {
        
    stage('Git') {
      steps {
          sh 'node -v'
          echo "${WORKSPACE}"
          
        git 'https://github.com/guthakondalu/nodejs.git'
      }
    }
     
    stage('Build') {
      steps {
        sh "npm install"
        
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
}