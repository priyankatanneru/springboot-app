pipeline {
    agent any

    environment {
        IMAGE = "chikky712/springboot-app"
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                sh 'echo passed'
                //git 'https://github.com/priyankatanneru/springboot-app.git'
            }
        }

        stage('Build & Test') {
            steps {
                script {
                    def mvnHome = tool 'Maven3'  // name must match your Maven tool in Jenkins
                    sh "${mvnHome}/bin/mvn clean package"
                }
            }
        }
	}
}
