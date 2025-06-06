pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '2d2620cf-4d7f-4125-b0f3-a9e5ab5e5a80'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')

    }

    stages {
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
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true               
                            }
                    }

                    steps {
                        sh '''
                        npm install serve
                        node_modules/.bin/serve -s build &
                        sleep 10
                        npx playwright test --reporter=line
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])

                        }
                    }
                }
            }

        }
        stage('deploy') {
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
                echo "Deploying to Netlify: $NETLIFY_SITE_ID"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --prod
        
            
                '''
            }
        }
        stage('Prod E2E test') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true               
                    }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://elaborate-pavlova-87b771.netlify.app'

        }

            steps {
                sh '''
                npx playwright test --reporter=line
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright prod E2E Report', reportTitles: '', useWrapperFileDirectly: true])

                }
            }
        }
    }       
    
}