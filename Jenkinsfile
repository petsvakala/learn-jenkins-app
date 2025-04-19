pipeline {
    agent any

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

    }


}
