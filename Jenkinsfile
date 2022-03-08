def testImage
//def stagingImage
//def productionImage
def REPOSITORY
def REPOSITORY_TEST
def RESPOSITORY_STAGING
def GIT_COMMIT_HASH
//def INSTANCE_ID
def ACCOUNT_REGISTRY_PREFIX
def S3_LOGS
def DATE_NOW
//def SLACK_TOKEN
//def CHANNEL_ID = "<YOUR_CHANNEL_ID>"



pipeline {
  agent any
  stages {
    stage("Set Up") {
      steps {
        echo "Logging into the private AWS Elastic Container Registry" 
        script {
          
          DATE_NOW = sh (script: "date +%Y%m%d", returnStdout: true)
          S3_LOGS = sh (script: "cat \$PWD/bucket_name", returnStdout: true)
          GIT_COMMIT_HASH = sh (script: "git log -n 1 --pretty=format:'%H'", returnStdout: true)
          REPOSITORY = sh (script: "cat \$PWD/repository_url", returnStdout: true)
          REPOSITORY_TEST = sh (script: "cat \$PWD/repository_test_url", returnStdout: true)
          REPOSITORY_STAGING = sh (script: "cat \$PWD/repository_staging_url", returnStdout: true)
          
          //ECR_REPO = sh (script: "cat \$PWD/ecr_repo", returnStdout: true)
         
         REPOSITORY_STAGING = REPOSITORY_STAGING.trim()
         REPOSITORY = REPOSITORY.trim()
         REPOSITORY_TEST = REPOSITORY_TEST.trim()
         S3_LOGS = S3_LOGS.trim()
         DATE_NOW = DATE_NOW.trim()
         ACCOUNT_REGISTRY_PREFIX = (REPOSITORY.split("/"))[0]
         //SLACK_TOKEN = SLACK_TOKEN.trim()
          
          
          sh """
          /bin/sh -e -c 'echo \$(aws ecr get-login-password --region us-east-1)  | docker login -u AWS --password-stdin $ACCOUNT_REGISTRY_PREFIX'
          """
        } 
      }
    }
    stage("Build Test Image") {
      steps {
        echo 'Start building the project docker image for tests' 
        script {
          testImage = docker.build("$REPOSITORY_TEST:$GIT_COMMIT_HASH", "-f ./Dockerfile.test .")
          testImage.push()
        } 
      }
    }
    stage("Run Unit Tests") {
      steps {
        echo 'Run unit tests in the docker image'
        script {
          def textMessage
          def inError
          try {
            testImage.inside('-v $WORKSPACE:/output -u root') {
              sh """
               
                cd $WORKSPACE/server
                npm run test:unit
                
                if test -d /output/unit ; then
                  rm -R /output/unit
                fi
                mv mochawesome-report /output/unit
              """
            }

            
            textMessage = "Commit hash: $GIT_COMMIT_HASH -- Has passed unit tests"
            inError = false

          } catch(e) {

            echo "$e"
          
            textMessage = "Commit hash: $GIT_COMMIT_HASH -- Has failed on unit tests"
            inError = true

          } finally {

            
            sh "aws s3 cp ./unit/ s3://$S3_LOGS/$DATE_NOW/$GIT_COMMIT_HASH/unit/ --recursive"
            
           

            if(inError) {
              
              error("Failed unit tests")
            }
          }
        }
      }
    }
    stage("Run Integration Tests") {
      steps {
        echo 'Run Integration tests in the docker image'
        script {
          def textMessage
          def inError
          try {
            testImage.inside('-v $WORKSPACE:/output -u root') {
              sh """
                cd $WORKSPACE/server
                npm run test:integration
                
                if test -d /output/integration ; then
                  rm -R /output/integration
                fi
                mv mochawesome-report /output/integration
              """
            }

            
            textMessage = "Commit hash: $GIT_COMMIT_HASH -- Has passed integration tests"
            inError = false

          } catch(e) {

            echo "$e"
            
            textMessage = "Commit hash: $GIT_COMMIT_HASH -- Has failed on integration tests"
            inError = true

          } finally {

            
            sh "aws s3 cp ./integration/ s3://$S3_LOGS/$DATE_NOW/$GIT_COMMIT_HASH/integration/ --recursive"

             
            if(inError) {
              
              error("Failed integration tests")
            }
          }
        }
      }
    }
  stage("Build Staging Image") {
      steps {
        echo 'Build the staging image for more tests'
        script {
          stagingImage = docker.build("$REPOSITORY_STAGING:$GIT_COMMIT_HASH")
          stagingImage.push()
        }
      }
    }
  }
}
