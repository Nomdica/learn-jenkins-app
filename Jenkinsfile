pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '77bf8678-d7e0-43da-bbea-8bf6c5a1060b'
    }

    stages {
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
        
        stage('Run Tests') {
            parallel {
                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                        ls -la 
                        test -f build/index.html
                        npm test
                        '''
                    }
                }
                stage('E2E') {
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
                npm install netlify-cli
                node_modules/.bin/netlify --version
                echo "Deploying to Production SITE ID : $NETLIFY_SITE_ID"
                '''
            }
        }
        
    }
    post {
        always {
            junit 'jest-results/junit.xml'
        }
    }
}
