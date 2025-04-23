pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'abccd-dfd4-dfdf0'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        AWS_DOCKER_REGISTRY = "566770890926.dkr.ecr.eu-central-1.amazonaws.com"
        AWS_DEFAULT_REGION = "eu-central-1"
        AWS_ECS_CLUSTER = "jenkins-cluster"
        AWS_ECS_SERVICE = "learnjenkins-app-service"
        AWS_ECS_TASK_DEFINITION = "jenkins-task-definition"
        APP_NAME = "myjenkinsapp"
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

        stage('Build Docker image for AWS') {
                agent {
                    docker {
                        image 'my-aws-cli'
                        reuseNode true
                        args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                    }
                }
                steps {
                    withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                        sh '''
                        docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION .
                        aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
                        docker push $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION
                        '''
                    }
                }
        }

        stage('AWS') {
            agent {
                docker {
                    image 'my-aws-cli'
                    reuseNode true
                    args "--entrypoint=''"
                }
            }
            environment {
                AWS_S3_BUCKET = 'learn-jenkinks-petsvakala'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                    aws --version
                    aws s3 ls
                    # echo "Hello S3!" > index.html
                    # aws s3 cp index.html s3://$AWS_S3_BUCKET/index.html
                    aws s3 sync build s3://$AWS_S3_BUCKET
                    '''
                }
            }
        }

        stage('Deploy to AWS using ECS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                    yum install jq -y
                    LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition.json | jq '.taskDefinition.revision')
                    echo $LATEST_TD_REVISION
                    aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE --task-definition $AWS_ECS_TASK_DEFINITION:$LATEST_TD_REVISION
                    aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE
                    '''
                }
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
                                image 'my-playwright'
                                reuseNode true
                            }
                        }
                            steps {
                                sh '''
                                    serve -s build &
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
                image 'my-playwright'
                reuseNode true
            }
        }
            steps {
                sh '''
                netlify --version
                echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                # netlify status
                # netlify deploy --dir=build --prod --json
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
                image 'my-playwright'
                reuseNode true
            }
        }
            steps {
                sh '''
                netlify --version
                echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                # node_modules/.bin/netlify status
                # netlify deploy --dir=build --prod --json > deploy-output.json
                # node-jq -r '.deploy-url' deploy-output.json
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
