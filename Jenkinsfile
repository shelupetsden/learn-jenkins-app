pipeline {
	agent any;

	environment {
		NETLIFY_SITE_ID = '8b7288d3-b874-4b32-a1b6-e7762cb6153a'
		NETLIFY_AUTH_TOKEN = credentials('netlify-token')
	}

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

		stage('Unit test') {
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

			post {
				always {
					junit 'jest-results/junit.xml'
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
					npm run build
				'''
			}
		}


		stage('Local E2E') {
			agent {
				docker {
					image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
					reuseNode true
				}
			}
			steps {
				sh '''
							npm install serve
							node_modules/.bin/serve -s build &
							sleep 10
							npx playwright test --reporter=html
						'''
			}

			post {
				always {
					publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Local HTML Report', reportTitles: '', useWrapperFileDirectly: true])
				}
			}
		}

		stage('Deploy') {
			agent {
				docker {
					image 'node:18-alpine'
					reuseNode true
				}
			}

			steps {
				sh '''
						 npm install netlify-cli@20.1.1
						 node_modules/.bin/netlify --version
						 echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
						 node_modules/.bin/netlify status
						 node_modules/.bin/netlify deploy --dir=build --prod --site "$NETLIFY_SITE_ID"
						'''
			}
		}


		stage('Prod E2E') {
			agent {
				docker {
					image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
					reuseNode true
				}
			}

			environment {
				CI_ENVIRONMENT_URL = 'https://learn-jenkins-cicd.netlify.app'
			}

			steps {
				sh '''
						npx playwright test --reporter=html
						echo $CI_ENVIRONMENT_URL
					'''
			}

			post {
				always {
					publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod HTML Report', reportTitles: '', useWrapperFileDirectly: true])
				}
			}
		}

	}
}