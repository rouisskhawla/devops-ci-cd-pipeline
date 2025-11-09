pipeline {
	agent any
	
	environment {
		DOCKER_REGISTRY = 'https://index.docker.io/v1/'
		REPO_NAME = 'khawlarouiss/devops-ci-cd-pipeline'
		DOCKER_CREDENTIALS_ID = 'dockerlogin'
		SSH_CREDENTIALS_ID = 'ssh-key-staging-vm'
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
    stage('Maven Test') {
      tools {
        maven 'Maven 3.9.11'
        jdk 'jdk17'
      }
      steps {
        sh 'mvn clean install -DskipTests'
		sh 'mvn test'
      }
    }
	stage('SonarQube Analysis') {
		tools {
			maven 'Maven 3.9.11'
			jdk 'jdk17'
		}
		steps {
			withSonarQubeEnv('sonar') {
				sh 'mvn sonar:sonar -Dsonar.projectKey=devops-ci-cd-pipeline'
			}
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
    stage('Build and Push Image to Docker Registry') {
    	steps {
    		script {
				docker.withRegistry("${DOCKER_REGISTRY}", "${DOCKER_CREDENTIALS_ID}") 
				{
					docker.build("${REPO_NAME}:latest", "-f Dockerfile .")
					docker.image("${REPO_NAME}:latest").push('latest')
				}
	        }
    	}
    }
	stage('Deploy') {
		steps {
			script {
				withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
					withEnv(["STAGING_VM=${env.STAGING_VM}"]) {
						sshagent([SSH_CREDENTIALS_ID]) {
							sh """
								scp -o StrictHostKeyChecking=no docker-compose.yml \${STAGING_VM}:devops-ci-cd-pipeline
								ssh -o StrictHostKeyChecking=no ${STAGING_VM} << EOF
								echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin
								cd devops-ci-cd-pipeline
								docker-compose -f docker-compose.yml pull
								docker-compose -f docker-compose.yml down
								docker-compose -f docker-compose.yml up -d
EOF
                                """
						}
					}
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
