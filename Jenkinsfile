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
    stage('Maven Install') {
      tools {
        maven 'Maven 3.9.11'
        jdk 'jdk17'
      }
      steps {
        sh 'mvn clean install -DskipTests'
      }
    }
    stage('Maven Package') {
      tools {
        maven 'Maven 3.9.11'
        jdk 'jdk17'
      }
      steps {
        sh 'mvn clean package -DskipTests'
      }
    }
    stage ('Maven Test') {
      tools { 
          maven 'Maven 3.9.11' 
          jdk 'jdk17' 
      }
      steps {
          sh 'mvn test'
      }
    }

  }

  post {
    always {
        cleanWs()
    }
  }
}
