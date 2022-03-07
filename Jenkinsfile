def testImage
def REPOSITORY
def REPOSITORY_TEST
def GIT_COMMIT_HASH
def ACCOUNT_REGISTRY_PREFIX
def ecr_repo




pipeline {
  agent any
  stages {
    stage("Set Up") {
      steps {
        echo "Logging into the private AWS Elastic Container Registry" 
        script {
          


          GIT_COMMIT_HASH = sh (script: "git log -n 1 --pretty=format:'%H'", returnStdout: true)
          REPOSITORY = sh (script: "cat \$PWD/repository_url", returnStdout: true)
          REPOSITORY_TEST = sh (script: "cat \$PWD/repository_test_url", returnStdout: true)
          
         

         REPOSITORY = REPOSITORY.trim()
         REPOSITORY_TEST = REPOSITORY_TEST.trim()
         
         ACCOUNT_REGISTRY_PREFIX = (REPOSITORY.split("/"))[0]
          
          
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
          testImage = docker.build("$REPOSITORY_TEST/$ecr_repo", "-f ./Dockerfile.test .")
          testImage.push()
        } 
      }
    }
  
  }
}
