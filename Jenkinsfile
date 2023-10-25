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
        JENKINS_API_TOKEN = credentials('JENKINS_API_TOKEN')

    }

    stages {

        stage("Cleanup Workspace") {
            steps {
            echo "Cleaning up workspace...wait for a sometimes..."
            cleanWs()
            echo "WOW...Workspace Cleaned Up..."
            }
            
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/hkalsait/jenkins-docker-k8s'
                echo "...Checking out our SCM code from above repository..."
            }
        }

        stage("Build Application") {
            steps{
                sh "mvn clean package"
                echo "...Our Application Build Successfully..."
            }

        }

        stage("Test Application") {
            steps{
                sh "mvn test"
                echo "...Our Application Testing Done Successfully..."
            }
        }

        stage("SonarQube Analysis") {
                steps{
                    script {
                        withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token'){
                            sh "mvn sonar:sonar"
                            echo "...SonarQube Analysis Done..."
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
        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                        echo "Great! Docker Image Build Successfully. Please check on dockerhub"
                    }
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                        echo "${DOCKER_USER} your docker Image has been pushed Successfully to dockerhub..."
                    }
                }
            }
        }

        /*stage("Trivy Scan") { #skipped due to memory issue...
            steps {
                script {
                   sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ashfaque9x/register-app-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
                }
            }
        }*/

        stage ('Cleanup Artifacts') {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                    echo "Artifact Cleanedup Succesfully"
                }
            }
        }

        stage("Trigger CD Pipeline") {
            steps {
                script {
                    echo "Triggering CD pipeline"
                    sh "curl -v -k --user great-success:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-54-173-106-113.compute-1.amazonaws.com:8080/job/gitops-app-pipeline/buildWithParameters?token=gitops-token'"
                    echo "Triggered CD pipeline inJenkins and you can check your ArgoCD dashboard, You application URL, status od K8s Pods and service..."
                    echo "Also your new build Image with new tag check in Docker as well as ArgoCD"
                    echo "...Thanks for choosing me..."
                }
            }
        }


    }

}
