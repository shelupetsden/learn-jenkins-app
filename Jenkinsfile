pipeline {
	agent any;

	environment {
		REACT_APP_VERSION = "1.0.$BUILD_ID"
		APP_NAME = 'learnjenkinsapp'
		AWS_DEFAULT_REGION = "eu-north-1"
		AWS_ECS_CLUSTER_PROD = 'LearnJenkinsApp-MyCluster-Prod'
		AWS_ECS_SERVICE_PROD = 'LearnJenkinsApp-MyService-Prod'
		AWS_ECS_TASK_DEFINITION_PROD = 'LearnJenkinsApp-MyTaskDefinition-Prod'
	}

	stages {
		stage('Build') {
			agent {
				docker {
					image 'node:18-alpine'
					reuseNode true
				}
			}

			steps {
				sh '''
				    ls -la
                    node --version
                    npm --version
                    npm ci
                    ls -la
					npm run build
				'''
			}
		}

		stage('Build Docker image') {
		    agent {
                docker {
                    image 'my-aws-cli'
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                    reuseNode true
                }
           }

           steps {
    	        sh '''
    	            docker build -t $APP_NAME:$REACT_APP_VERSION .
    	        '''
    	   }
      	}

		stage('Deploy AWS') {
                agent {
                    docker {
                        image 'my-aws-cli'
                        args "-u root --entrypoint=''"
                        reuseNode true
                    }
                }


               steps {
                    withCredentials([usernamePassword(credentialsId: 'my-aws-s3', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                        sh '''
                            aws --version
                            yum install jq -y
                            LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                            echo $LATEST_TD_REVISION
                            aws ecs update-service --cluster $AWS_ECS_CLUSTER_PROD --service $AWS_ECS_SERVICE_PROD --task-definition $AWS_ECS_TASK_DEFINITION_PROD:$LATEST_TD_REVISION
                            aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER_PROD --services $AWS_ECS_SERVICE_PROD
                            '''
                    }
               }

        }
    }
}