pipeline {
    agent { label 'Jenkins-Agent' }
    tools {
        jdk 'Java17'
        maven 'Maven3'
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
    }
}
