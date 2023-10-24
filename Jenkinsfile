pipeline {

    agent {
        label 'Jenkins-Agent'
    }

    tools {
        jdk 'Java17'
        maven 'Maven3'
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
                git branch: 'main', credentialId: 'github', url: 'https://github.com/hkalsait/jenkins-docker-k8s'
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
        
    }

}