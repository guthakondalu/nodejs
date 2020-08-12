pipeline {
  agent any

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

          git credentialsId: 'e19ff800-0e8b-4b9b-8a15-8495d818cd5d', url: 'https://github.com/guthakondalu/nodejs.git'     
      }  
    }
     
    stage('Test') {
      steps {
        sh "npm test"
      }
    }
    
    stage('Commit Code Coverage Metrics to GIT') {
      steps {
         sh """
            git add -f coverage
            git commit -m "Code coverage metrics added from build -  $BUILD_NUMBER"
            git push origin master
        """
      }
    }
  }

post {
        success {
               	cleanWs()
                echo 'Successfully completed the Build'
            }
        
        failure {
            echo 'Triggering email'
           
        }
    }


}