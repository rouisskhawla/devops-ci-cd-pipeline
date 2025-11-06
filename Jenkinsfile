pipeline {
  agent any
  
  stages {
    
    stage ('Clone Github Repository'){
      steps {
        checkout([
          $class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[ 
            url: 'git@github.com:rouisskhawla/devops-ci-cd-pipeline.git',
            credentialsId: 'github'
          ]]
        ])
      }
    }
    
  }
  
  post {
    always {
        cleanWs()
    } 
  }
  
}
