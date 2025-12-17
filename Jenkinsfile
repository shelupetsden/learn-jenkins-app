pipeline {
	agent any

	stages {
		stage('Install') {
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
				'''
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
					npm run build
				'''
			}
		}

		stage('Test') {
			agent {
				docker {
					image 'node:18-alpine'
					reuseNode true
				}
			}

			steps {
				sh '''
				test -f build/index.html
				npm test
			'''

			}
		}
	}

	post {
		always {
			junit 'test-results/junit.xml'
		}
	}
}