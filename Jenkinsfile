pipeline {
    agent any

    stages {
        /*stage('build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true               
                    }
            }
            steps {
                sh '''
                ls -la
                npm --version
                node --version
                npm ci
                npm run build
                '''
            }
        }
        */
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
                }        
            }

        }
        
    }       
    post {
        always {
            junit 'jest-results/junit.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])

        }

    
    }
}