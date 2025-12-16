pipeline {

	agent {
		docker {
			image 'node:18-alpine'
			reuseNode true
		}
	}

	stages {
		stage('Install') {
			steps {
				sh '''
					echo -------Install----------
					ls -la
					node --version
					npm --version
					npm ci
					ls -la
				'''
			}
		}

		stage('Build') {
			steps {
				sh '''
				echo -------Build----------
					npm run build
				'''
			}
		}

		stage('Test') {
			steps {
				sh '''
				echo -------TEST----------
				test -f build/index.html
				npm test
			'''

			}
		}
	}
}