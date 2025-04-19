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
        }
    }

    post {
    always {
        junit 'test-results/junit.xml'
        }
    }
}
