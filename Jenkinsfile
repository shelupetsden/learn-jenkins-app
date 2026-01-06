pipeline {
	agent any;

	environment {
		REACT_APP_VERSION = "1.0.$BUILD_ID"
		AWS_DEFAULT_REGION = "eu-north-1"
	}

	stages {

	    stage('Deploy AWS') {
               agent {
               	    docker {
                	    image 'amazon/aws-cli'
               	        args "--entrypoint=''"
               	        reuseNode true
                    }
               }

               steps {
                   withCredentials([usernamePassword(credentialsId: 'my-aws-s3', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                       sh '''
                             aws --version
                             aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json
                             aws ecs update-service --cluster LearnJenkinsApp-MyCluster-Prod --service LearnJenkinsApp-MyService-Prod --task-definition LearnJenkinsApp-MyTaskDefinition-Prod:2
                         '''

                   }
               }
        }

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
	}
}