pipeline {
  agent any
  stages {
    stage (clone github repository){
      steps {
        cleanWs()
        checkout([$class: 'GitSCM', branches: [[name: '/*main']], userRemoteConfigs: [[ url: 'git@github.com:rouisskhawla/devops-ci-cd-pipeline.git', credentialsId: 'github' ]]])
      }
    }
  }
}
