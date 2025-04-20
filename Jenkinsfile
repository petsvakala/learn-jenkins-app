pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'abccd-dfd4-dfdf0'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = '1.2.3'
        }

    stages {
        // This is a comment
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
                    npm run build
                    ls -la
                '''
            }
        }
        /*
        block comment line 1
        block comment line 2
        */
        stage('Run Tests') {
                parallel {
                        stage('Test') {
                        agent {
                            docker {
                                image 'node:18-alpine'
                                reuseNode true
                            }
                        }
                            steps {
                                sh '''
                                    echo "Test stage"
                                    find build -name index.html
                                    npm test
                                '''
                            }
                            post {
                                always {
                                    junit 'jest-results/junit.xml'
                                    }
                                }
                        }

                        stage('E2E') {
                        agent {
                            docker {
                                image 'mcr.microsoft.com/playwright:v1.52.0-noble'
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
                                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PlayWright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                                    }
                                }

                        }
                }
        }

        stage('Deploy Staging') {
        agent {
            docker {
                image 'node:18-alpine'
                reuseNode true
            }
        }
            steps {
                sh '''
                npm install netlify-cli
                node_modules/.bin/netlify --version
                echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                # node_modules/.bin/netlify status
                # node_modules/.bin/netlify deploy --dir=build --prod --json
                # comment
                '''
            }
        }

//         stage('Approval') {
//                 steps {
//                 timeout(time: 120, unit: 'SECONDS') {
//                     input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
//                     }
//                 }
//         }

        stage('Deploy Prod') {
        agent {
            docker {
                image 'node:18-alpine'
                reuseNode true
            }
        }
            steps {
                sh '''
                npm install netlify-cli node-jq
                node_modules/.bin/netlify --version
                echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                # node_modules/.bin/netlify status
                # node_modules/.bin/netlify deploy --dir=build --prod --json > deploy-output.json
                # node_modules/.bin/node-jq -r '.deploy-url' deploy-output.json
                '''
                /*
                script {
                    env.STAGING_URL = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
                }
                */
            }
        }


        stage('PROD E2E') {
                environment {
                    CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
                }
                steps {
                    sh '''
                        echo "dummy prod E2E"
                    '''
                }
        }
    }

}
