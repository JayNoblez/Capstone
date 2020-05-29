pipeline {
	agent any
	stages {

		stage('Linting') {
			steps {
				sh 'tidy -q -e *.html'
			}
		}
		
		stage('Build Docker Image') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'DockerHubCred', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						sudo -n docker build -t jaynoblez/jkcapstone .
					'''
				}
			}
		}

		stage('Push Image To Dockerhub') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'DockerHubCred', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						sudo -n docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD 
						sudo -n docker push jaynoblez/jkcapstone
					'''
				}
			}
		}	
              stage('Set current kubectl context') {
			steps {
				withAWS(region:'eu-central-1', credentials:'AWSJenkins') {
					sh '''
						sudo kubectl config get-contexts
						sudo kubectl config use-context arn:aws:eks:eu-central-1:842187592768:cluster/JK8capstone
					'''
				}
			}
		}

		stage('Deploy blue container') {
			steps {
				withAWS(region:'eu-central-1', credentials:'AWSJenkins') {
					sh '''
						kubectl apply -f ./blue-controller.json
					'''
				}
			}
		}
}
}
