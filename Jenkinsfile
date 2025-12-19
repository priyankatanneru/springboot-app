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
                sh 'mvn clean test'
            }
        }

        stage('SonarQube') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh """
                docker build -t $IMAGE:$TAG .
                docker push $IMAGE:$TAG
                """
            }
        }

        stage('Update Manifests Repo') {
            steps {
                git credentialsId: 'github-creds',
                    url: 'https://github.com/priyankatanneru/springboot-k8s-manifests.git'

                sh """
                sed -i 's/tag:.*/tag: "$TAG"/' helm/values.yaml
                git add .
                git commit -m "Update image to $TAG"
                git push
                """
            }
        }
    }
}

