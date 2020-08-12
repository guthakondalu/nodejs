@Library('pileline-shared-library')_

def releaseBranch = 'release'
def jobnameparts = JOB_NAME.tokenize('/') as String[]
def jobconsolename = jobnameparts[1]
def workdir = "${JOB_BASE_NAME}".replace("/", "_").replace("%2F", "-")

pipeline {
  /*
   * Run everything on an existing agent configured with a label 'docker'.
   * This agent will need docker, git and a jdk installed at a minimum.
   */
    agent {
    docker {
        label 'node1'
        image 'lambci/lambda:build-nodejs12.x'
      	customWorkspace "/home/ec2-user/jenkins/node1/workspace/${jobconsolename}-${workdir}"
        args '-u root'
        reuseNode true
      }
    }
  options {
    disableConcurrentBuilds() 
    timestamps()
    ansiColor('xterm')
    buildDiscarder(logRotator(numToKeepStr: '5', daysToKeepStr: '10'))
  }
  environment {
    CD_APP_NAME = "schoolnet-newss"
    PUPPETEER_SKIP_CHROMIUM_DOWNLOAD = 1
    npm_config_cache = 'npm-cache'
    HOME="$WORKSPACE"
    }
    stages {
        stage('build') {
            steps {
                setAuthorLinux()
                script {
                  def tempname = ""
                  env.featureStageName = "dev"
                  if (workdir.contains('-')) {
                    tempname = workdir.split("-")
                    env.revision = "${tempname[1]}.${env.BUILD_ID}"
                  }
                  else {
                    tempname = workdir
                    env.revision = "${tempname}.${env.BUILD_ID}"
                  }
                }
                sh """
                    echo "Tool versions"
                    node --version
                    npm --version
                    
                    echo PUPP SKIP: \$PUPPETEER_SKIP_CHROMIUM_DOWNLOAD
                    echo HOME: \$HOME

                    rm -rf pkg
                    rm -rf package*.zip
                    chmod 0777 -R *
                    chmod 0777 -R .fonts/
                    npm install
                    npm install serverless-plugin-include-dependencies --save-dev
                """
            }
        }
        stage('Release-stage') {
            steps {
                script{
                    if (env.BRANCH_NAME.contains(releaseBranch)) {
                        currentBuild.displayName = "#Release:${env.revision}"
                    }
                    else{
                        currentBuild.displayName = "#${env.revision}"	
                    }
                    lock("${env.CD_APP_NAME}-${featureStageName}") {
                        /*
                        * Multiline strings can be used for larger scripts. It is also possible to put scripts in your shared library
                        * and load them with 'libaryResource'
                        */
                        sh """
                            echo '************* deploy ${CD_APP_NAME} version ${revision} to ${featureStageName} *************'
                            npx serverless deploy --verbose --stage ${featureStageName}
                        """
                    }
                }
            }
        }
    }
    post {
        success {
            script{
                if (env.BRANCH_NAME.contains(releaseBranch)){
                    s3Upload acl: 'Private', bucket: 'schoolnet-artifacts', cacheControl: '', excludePathPattern: '', file: '.serverless/newss.zip', metadatas: [''], path: "schoolnet-lambda/${workdir}-${env.BUILD_ID}/", sseAlgorithm: '', workingDir: ''
                    s3Upload acl: 'Private', bucket: 'schoolnet-artifacts', cacheControl: '', excludePathPattern: '', file: 'serverless.yml', metadatas: [''], path: "schoolnet-lambda/${workdir}-${env.BUILD_ID}/", sseAlgorithm: '', workingDir: ''
                }
                jiraUpdate()
              	cleanWs()
            }
        }
        failure {
            echo 'Triggering email'
            //emailfailure()
        }
    }
}