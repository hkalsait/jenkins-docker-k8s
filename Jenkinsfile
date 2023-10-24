pipeline {

    agent {
        label 'Jenkins-Agent'
    }

    tools {
        jdk 'Java17'
        maven 'Maven3'
    }

    environment {
        APP_NAME = "cicd-pipeline-app"
        RELEASE_VERSION = "1.0.0"
        DOCKER_USER = "hkalsait"
        DOCKER_PASS = "dockerhub"
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE_VERSION}-${BUILD_NUMBER}"
    }

    stages {

        stage("Cleanup Workspace") {
            steps {
            echo "Cleaning up workspace..."
            cleanWs()
            }
            
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/hkalsait/jenkins-docker-k8s'
                echo "Checking out our SCM code from above repository..."
            }
        }

        stage("Build Application") {
            steps{
                sh "mvn clean package"
            }

        }

        stage("Test Application") {
            steps{
                sh "mvn test"
            }
        }

    stage("SonarQube Analysis") {
            steps{
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token'){
                    sh "mvn sonar:sonar"
                }
                }
            }
        }

    stage("Quality gate"){
        steps{
            script{
                waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
            }
        }
    }
      /*  -stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry(' ',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }*/
        
    }

}
