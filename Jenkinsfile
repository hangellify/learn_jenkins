pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'fa642489-83cc-4f38-bcdf-9490ef7bb960'
        NETLIFY_AUTH_TOKEN = credentials('NETLIFY_TOKEN')
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                    args '-u root:root'
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    rm -rf node_modules package-lock.json.lock
                    export npm_config_cache=/tmp/.npm-cache
                    npm ci --cache $npm_config_cache
                    npm run build
                    ls -la
                '''
            }
        }

//        stage ('Run Tests') {
//            parallel {
//                 stage('Unit Tests') {
//                    agent {
//                        docker {
//                            image 'node:18-alpine'
//                            reuseNode true
//                        }
//                    }
//                    steps {
//                        sh '''
//                            test -f build/index.html
//                            npm run test
//                        '''
//                    }
//
//                    post {
//                        always {
//                            junit 'jest-results/junit.xml'
//                        }
//                    }
//                }
//                stage('E2E') {
//                    agent {
//                        docker {
//                            image 'mcr.microsoft.com/playwright:v1.58.2-noble'
//                            reuseNode true
//                        }
//                    }
//                    steps {
//                        sh '''
//                            export npm_config_cache=/tmp/.npm-cache
//                            mkdir -p $npm_config_cache
//
//                            npm install serve --cache $npm_config_cache
//                            node_modules/.bin/serve -s build &
//
//                            sleep 10
//
//                            npx playwright test --reporter=html
//                        '''
//                    }
//                    post {
//                        always {
//                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
//                        }
//                    }
//                }
//            }
//        }

        stage('Deploy') {
            // No Docker: runs on host where Jenkins user exists in /etc/passwd (fixes uv_os_get_passwd)
            agent {
                docker {
                   image 'node:18-alpine'
                   reuseNode true
                   args '-u root:root'
                }
            }
            steps {
                sh '''
                    export npm_config_cache=/tmp/.npm-cache
                    mkdir -p $npm_config_cache

                    npm install netlify-cli@20.1.1 --cache $npm_config_cache
                    node_modules/.bin/netlify --version

                    echo "Deployed to production: $NETLIFY_SITE_ID"

                    node_modules/.bin/netlify status
                '''
            }
        }

    }

}