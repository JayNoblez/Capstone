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
				withAWS(region:'us-east-2', credentials:'aws-credentials') {
					sh '''
						sudo -s
						kubectl config get-contexts
						kubectl config use-context  arn:aws:eks:us-east-2:493716690734:cluster/capstonecluster
					'''
				}
			}
		}

		stage('Deploy blue container') {
			steps {
				withAWS(region:'us-east-2', credentials:'aws-credentials') {
					sh '''
						kubectl apply -f ./blue-controller.json
					'''
				}
			}
		}
}
}
