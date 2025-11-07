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
    stage('Docker Login'){
      steps {
        script {
          withCredentials([
            usernamePassword(credentialsId: 'dockerlogin', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')
          ]) {
              sh 'docker login -u="$DOCKER_USERNAME" -p="$DOCKER_PASSWORD"'
              }
        }
      }
    }  
    stage('Build & Push Image '){
      steps {
        script {
          docker.withRegistry('https://registry.hub.docker.com', 'dockerlogin') {
            def customImage = docker.build('khawlarouiss/devops-ci-cd-pipeline:latest', '.')
            customImage.push()
          }
        }
      }
    }
    
  }

  post {
    always {
        cleanWs()
    }
  }
}
