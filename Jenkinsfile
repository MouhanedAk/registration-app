pipeline {
    agent { label 'Jenkins-Agent' }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
	    APP_NAME = "register-app-pipeline"
            RELEASE = "1.0.0"
            DOCKER_USER = "mouhanedakermi"
            DOCKER_PASS = 'dockerhub'
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        stage("Checkout from SCM"){
                steps {
                    git branch: 'master', credentialsId: 'github', url: 'https://github.com/MouhanedAk/registration-app'
                }
        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }

       }

       stage("Test Application"){
           steps {
                 sh "mvn test"
           }
       }

       stage("SonarQube Analysis"){
           steps {
	           script {
		        withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') { 
                        sh "mvn sonar:sonar"
		        }
	           }	
           }
       }

       stage("Quality Gate"){
           steps {
               script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }	
            }

        }
       stage("Build & Push Docker Image") {
           steps {
               script {
            // Build the Docker image
            sh """
            docker build -t ${IMAGE_NAME} .
            """

            // Log in to Docker Hub
            sh """
            echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin
            """

            // Push the Docker image with the specific tag
            sh """
            docker tag ${IMAGE_NAME} ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG}
            docker push ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG}
            """

            // Optionally, push the 'latest' tag
            sh """
            docker tag ${IMAGE_NAME} ${DOCKER_USER}/${IMAGE_NAME}:latest
            docker push ${DOCKER_USER}/${IMAGE_NAME}:latest
            """
               }
           }
       }
    }
}
