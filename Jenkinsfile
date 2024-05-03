pipeline {
    agent { label 'Jenkins-Agent' }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }

    environment { 
	    APP_NAME = "register-app"
	    RELEASE = "1.0.0"
	    DOCKER_USER = "arpit3323"
	    DOCKER_PASS = "dockerhub"
	    IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
    stages {
          stage ("Cleanup Workspace") {
                steps {
                 cleanWs() 
                }
            
          }
        stage ("Git Checkout") {
               steps {
                  git branch: 'main', credentialsId: 'github' , url: 'https://github.com/arpit3323/Register-app'
               }
        }

        stage ("Build Application") {
               steps {
                  sh "mvn clean package"
               }
        }

      stage ("Test Application") {
               steps {
                  sh "mvn test"
               }
        }

	stage ("SonarQube Analysis") {
               steps {
                       script {
		             withSonarQubeEnv(credentialsId: 'jenkins-sonar-cube'){
			      sh "mvn sonar:sonar"

			     }

		       }
               }

          }
	    stage ("Quality Gate") {
               steps {
                       script {
		             waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonar-cube'
		       
		       }
               }

          }

	  stage ("Build & Push Docker Image") {
               steps {
                       script {
		             docker.withRegistry('',DOCKER_PASS){
				     docker_image = docker.build "${IMAGE_NAME}"
			     }

			     docker.withRegistry('',DOCKER_PASS){
				docker_image.push("${IMAGE_NAME}")
				docker_image.push('v1.0')
				     
			     }

			
		       
		       }
               }

          }
    }
}
