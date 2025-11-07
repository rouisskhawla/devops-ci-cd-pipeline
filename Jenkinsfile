pipeline {
  agent any
  
  environment {
      DOCKER_REGISTRY = 'https://index.docker.io/v1/'
      REPO_NAME = 'khawlarouiss/devops-ci-cd-pipeline'
      DOCKER_CREDENTIALS_ID = 'dockerlogin'
  }

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
	  stage('Docker Login') {
      steps {
        script {
          withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
            sh """
                echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin
            """
          }
        }
      }
    }
    stage('Build Docker Image') {
      steps {
        script {
            docker.build("${REPO_NAME}:latest", "-f Dockerfile .")
        }
      }
    }
    stage('Push Image to Docker Registry') {
      steps {
        script {
          docker.withRegistry("${DOCKER_REGISTRY}", "${DOCKER_CREDENTIALS_ID}") {
            docker.image("${REPO_NAME}:latest").push('latest')
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
