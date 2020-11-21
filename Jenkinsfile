pipeline {
	agent any
	stages {

		stage('Lint HTML') {
			steps {
				sh 'tidy -q -e *.html'
			}
		}
		
		stage('Build Docker Image') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'mohamedesamanas', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						sudo -S docker build -t mohamedesamanas/udacity_capstone . </home/ubuntu/pass.txt
					'''
				}
			}
		}

		stage('Push Image To Dockerhub') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'mohamedesamanas', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						sudo docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD -S
						sudo docker push mohamedesamanas/udacity_capstone -S
					'''
				}
			}
		}

		stage('Set current kubectl context') {
			steps {
				withAWS(region:'us-east-2', credentials:'capstone_cred') {
					sh '''
						kubectl config use-context arn:aws:cloudformation:us-east-2:549195752255:stack/eksctl-capstonecluster-cluster/5f8e11c0-2ba0-11eb-b748-06e35090f892
					'''
				}
			}
		}

		stage('Deploy blue container') {
			steps {
				withAWS(region:'us-east-2', credentials:'capstone_cred') {
					sh '''
						kubectl apply -f ./blue-controller.json
					'''
				}
			}
		}

		stage('Deploy green container') {
			steps {
				withAWS(region:'us-east-2', credentials:'capstone_cred') {
					sh '''
						kubectl apply -f ./green-controller.json
					'''
				}
			}
		}

		stage('Redirect to blue') {
			steps {
				withAWS(region:'us-east-2', credentials:'capstone_cred') {
					sh '''
						kubectl apply -f ./blue-service.json
					'''
				}
			}
		}

		stage('Waiting user approval') {
            steps {
                input "Would you like to Redirect traffic to green?"
            }
        }

		stage('Redirect to green') {
			steps {
				withAWS(region:'us-east-2', credentials:'capstone_cred') {
					sh '''
						kubectl apply -f ./green-service.json
					'''
				}
			}
		}

	}
}
