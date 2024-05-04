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
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
    }

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs() 
            }   
        }
        stage("Git Checkout") {
            steps {
                git branch: 'main', credentialsId: 'github' , url: 'https://github.com/arpit3323/Register-app'
            }
        }
        stage("Build Application") {
            steps {
                sh "mvn clean package"
            }
        }
        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }
        stage("SonarQube Analysis") {
            steps {
                script {
                withSonarQubeEnv(credentialsId: 'jenkins-sonar-cube') {
                    sh "mvn sonar:sonar"
                }
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonar-cube'
            }
            }
        }
        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('', DOCKER_PASS) {
                        def docker_image = docker.build("${IMAGE_NAME}")
                        docker_image.push("latest")
                    }
                }
            }
        }

        stage {
            steps {
                script {
                    sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image arpit3323/register-app:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
                }
            }
        }

        stage {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }

        stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user clouduser:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-13-59-248-215.us-east-2.compute.amazonaws.com:8080/job/gitops-register-app-cd/buildWithParameters?token=gitops-token'"
                }
            }
       }
    }
}
