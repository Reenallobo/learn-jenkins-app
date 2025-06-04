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
        stage('test') {
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
            stage('E2E test') {
                agent {
                    docker {
                        image 'mcr.microsoft.com/playwright:v1.52.0-noble'
                        reuseNode true               
                        }
                }

            steps {
                sh '''
                npm install -g serve
                serve -s build
                npx playwright test
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
}