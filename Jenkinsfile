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
        stage('SonarQube') {
            steps {
                script {
                    def mvnHome = tool 'Maven3'
                    withSonarQubeEnv('sonarqube') {
                        sh """
                		${mvnHome}/bin/mvn sonar:sonar \
                  			-Dsonar.projectKey=springboot-app
                		"""                
                    }
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build & Push') {
           steps {
              withCredentials([usernamePassword(
              credentialsId: 'dockerhub-creds',
              usernameVariable: 'DOCKER_USER',
              passwordVariable: 'DOCKER_PASS'
              )]) {
                  sh """
                  echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                  docker build -t $IMAGE:$TAG .
                  docker push $IMAGE:$TAG
                  docker logout
                  """
             }
          }
        }

        stage('Update Manifests Repo') {
            steps {
                git credentialsId: 'github-creds',
                url: 'https://github.com/priyankatanneru/springboot-k8s-manifests.git',
                branch: 'main'

                withCredentials([usernamePassword(
                credentialsId: 'github-creds', 
                usernameVariable: 'GIT_USER', 
                passwordVariable: 'GIT_PASS'
                )]) {
                    sh """
                        git config user.email "tannerupriyanka712@gmail.com"
                	git config user.name "priyankatanneru"
                	sed -i 's/tag:.*/tag: ${TAG}/' helm/values.yaml
                	git add .
                	git commit -m "Update image to $TAG" || echo "No changes to commit"
	                git push https://$GIT_USER:$GIT_PASS@github.com/priyankatanneru/springboot-k8s-manifests.git main
        	    """
                }
             }
         }
     }
}

