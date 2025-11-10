pipeline {
	agent any
	
	environment {
		DOCKER_REGISTRY = 'https://index.docker.io/v1/'
		REPO_NAME = 'khawlarouiss/devops-ci-cd-pipeline'
		DOCKER_CREDENTIALS_ID = 'dockerlogin'
		SSH_CREDENTIALS_ID = 'ssh-key-staging-vm'
		NEXUS_CREDENTIALS = 'nexus-credentials'
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
			catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
				withSonarQubeEnv('sonar') {
					sh 'mvn sonar:sonar -Dsonar.projectKey=devops-ci-cd-pipeline'
				}
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
	stage('Maven Deploy To Nexus') {
		tools {
			maven 'Maven 3.9.11'
			jdk 'jdk17'
		}
		steps {
			input "Do you want to deploy Artifact to Nexus?"
			withCredentials([usernamePassword(
				credentialsId: env.NEXUS_CREDENTIALS,
				usernameVariable: 'NEXUS_USER',
				passwordVariable: 'NEXUS_PASS'
			)]) {
				configFileProvider([configFile(fileId: 'maven-settings', variable: 'MAVEN_SETTINGS')]) {
					sh "mvn clean deploy -s $MAVEN_SETTINGS -DaltDeploymentRepository=nexus::default::$NEXUS_RELEASE_URL"
				}
			}
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
			input "Do you want to build and push the Docker image?"
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
			input "Do you want to deploy to staging environment?"
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
