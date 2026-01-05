pipeline {
	agent any;

	environment {
		NETLIFY_SITE_ID = '8b7288d3-b874-4b32-a1b6-e7762cb6153a'
		NETLIFY_AUTH_TOKEN = credentials('netlify-token')
		REACT_APP_VERSION = "1.0.$BUILD_ID"
	}

	stages {
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
				    ls -la
                    node --version
                    npm --version
                    npm ci
                    ls -la

					npm run build
				'''
			}
		}


		stage('Local E2E') {
			agent {
				docker {
					image 'my-playwright'
					reuseNode true
				}
			}
			steps {
				sh '''
						serve -s build & sleep 10
						npx playwright test --reporter=html
					'''
			}

			post {
				always {
					publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Local HTML Report', reportTitles: '', useWrapperFileDirectly: true])
				}
			}
		}

		stage('Deploy staging') {
			agent {
				docker {
					image 'my-playwright'
					reuseNode true
				}
			}

			steps {
				sh '''
					#deploy
					 netlify --version
					 echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
					 netlify status
					 netlify deploy --dir=build --site "$NETLIFY_SITE_ID" --json > deploy-response.json
				'''

				script {
					env.STAGING_DEPLOY_URL = sh(
						script: "node-jq -r '.deploy_url' deploy-response.json",
						returnStdout: true
					).trim()
					echo "Staging deploed URL: ${env.STAGING_DEPLOY_URL}"
				}

				sh '''
					#wait after deploy
					echo "Waiting for staging to be reachable: $STAGING_DEPLOY_URL"
    				for i in $(seq 1 30); do
    				if curl -fsS "$STAGING_DEPLOY_URL" >/dev/null; then echo "Stage is reachable."
    				break
    				fi
      				echo "Not ready yet... attempt $i/30"
      				sleep 5
    				done

					#E2E test
					echo "Using staging URL: $STAGING_DEPLOY_URL"
    				CI_ENVIRONMENT_URL="$STAGING_DEPLOY_URL" npx playwright test --reporter=html
  				'''
			}

			post {
				always {
					publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging HTML Report', reportTitles: '', useWrapperFileDirectly: true])
				}
			}
		}

// 		stage('Approval') {
// 			steps {
// 				timeout(time: 15, unit: "HOURS") {
// 					input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
// 				}
//
// 			}
//
// 		}
//

		stage('Deploy Prod') {
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
						echo "Using prod URL: $CI_ENVIRONMENT_URL"

						#deploy
						netlify --version
						echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
						netlify status
						netlify deploy --dir=build --prod --site "$NETLIFY_SITE_ID"

						#wait after deploy
						echo "Waiting for prod to be reachable: $CI_ENVIRONMENT_URL"
    					for i in $(seq 1 30); do
    					if curl -fsS "$CI_ENVIRONMENT_URL" >/dev/null; then echo "Prod is reachable."
    					break
    					fi
      					echo "Not ready yet... attempt $i/30"
      					sleep 5
    					done

						#E2E test
						npx playwright test --reporter=html
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