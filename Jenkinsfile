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
						sudo -s
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
 stage('Deploy Green Container') {
                        steps {
                                withAWS(region:'eu-central-1', credentials:'AWSJenkins') {
                                        sh '''
                                                kubectl apply -f ./green-controller.json
                                        '''
                                }
                        }
                }

                stage('Create the service in the cluster, redirect to blue') {
                        steps {
                                withAWS(region:'eu-central-1', credentials:'AWSJenkins') {
                                        sh '''
                                                kubectl apply -f ./blue-service.json
                                        '''
                                }
                        }
                }

                stage('Wait for User to Approve') {
             steps {
                input "Ready to redirect traffic to green?"
            }
        }

                stage('Create the Service in the Cluster, Redirect to green') {
                        steps {
                                withAWS(region:'eu-central-1', credentials:'AWSJenkins') {
                                     sh '''
                                                kubectl apply -f ./blue-service.json
                                        '''
                                }
                        }
                }

                stage('Wait for User to Approve') {
            steps {
                input "Ready to redirect traffic to green?"
            }
        }

                stage('Create the Service in the Cluster, Redirect to green') {
                        steps {
                                withAWS(region:'eu-central-1', credentials:'AWSJenkins') {
                                        sh '''
                                                kubectl apply -f ./green-service.json
                                        '''
                                }
                        }
                }

}
}
