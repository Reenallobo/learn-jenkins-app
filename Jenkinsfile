pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '2d2620cf-4d7f-4125-b0f3-a9e5ab5e5a80'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.${BUILD_NUMBER}"
    }

    stages {
        stage('Docker') {
            steps {
                sh 'docker build -t myplaywright-image .'
            }
        }
        
        stage('build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true               
                    }
            }
            steps {
                sh '''
                echo "git polling introduced"
                ls -la
                npm --version
                node --version
                npm ci
                npm run build
                '''
            }
        }
        
        stage('test'){
            parallel{
                stage('unit test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true               
                        }
                    }

                    steps {
                        sh '''
                        #echo "Test stage"
                        #test -f build/index.html
                        npm test
                        '''
                    }

                    post {
                        always {
                            junit 'jest-results/junit.xml'
                            

                        }                  
                    }
                }

  
                stage('E2E test') {
                    agent {
                        docker {
                            image 'myplaywright-image'
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
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Local E2E Report', reportTitles: '', useWrapperFileDirectly: true])

                        }
                    }
                }
            }

        }
        
        stage('deploy staging') {
            agent {
                docker {
                    image 'myplaywright-image'
                    reuseNode true               
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "YET_TO_BE_SET"

            }

            steps {
                sh '''
                netlify --version
                echo "Deploying to staging Netlify: $NETLIFY_SITE_ID"
                netlify status
                netlify deploy --dir=build --json > deploy-output.json
                CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
                npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E Report', reportTitles: '', useWrapperFileDirectly: true])

                }
            }
        }

        stage('deploy prod') {
            agent {
                docker {
                    image 'myplaywright-image'
                    reuseNode true               
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://elaborate-pavlova-87b771.netlify.app'

            }

            steps {
                sh '''
                node --version
                netlify --version
                echo "Deploying to prod Netlify: $NETLIFY_SITE_ID"
                netlify status
                netlify deploy --dir=build --prod
                npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'prod E2E Report', reportTitles: '', useWrapperFileDirectly: true])

                }
            }
        }
    }       
    
}