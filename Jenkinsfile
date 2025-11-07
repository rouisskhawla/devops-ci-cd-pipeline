pipeline {
  agent any

  stages {
    stage('Clone Github Repository') {
      steps {
        checkout([
          $class: 'GitSCM',
          branches: [[name: 'main']],
          userRemoteConfigs: [[
            url: 'git@github.com:rouisskhawla/devops-ci-cd-pipeline.git',
            credentialsId: 'github'
          ]]
        ])
      }
    }
    stage('Maven install') {
      tools {
        maven 'Maven 3.9.5'
        jdk 'jdk17'
      }
      steps {
        sh 'mvn install -DskipTests'
      }
    }
    stage('Maven clean package') {
      tools {
        maven 'Maven 3.9.5'
        jdk 'jdk17'
      }
      steps {
        sh 'mvn clean package'
      }
    }
  }

  post {
    always {
        cleanWs()
    }
  }
}
