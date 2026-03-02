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
                    image 'my-playwright'
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

        stage ('Run Tests') {
            parallel {
                 stage('Unit Tests') {
                    agent {
                        docker {
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            test -f build/index.html
                            npm run test
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
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            // No Docker: runs on host where Jenkins user exists in /etc/passwd (fixes uv_os_get_passwd)
            agent {
                docker {
                   image 'my-playwright'
                   reuseNode true
                   args '-u root:root'
                }
            }
            steps {
                sh '''
                    echo "Deployed to production: $NETLIFY_SITE_ID"

                    netlify status
                    netlify deploy --dir=build --prod --json > deploy-output.json
                    node-jq -r '.deploy_url' deploy-output.json
                '''
                script {
                    env.STAGE_URL = sh(script: "node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
                }
            }
        }

        stage('Stage E2E') {
                    agent {
                        docker {
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    environment {
                        CI_ENVIRONMENT_URL = "${env.STAGE_URL}"
                    }

                    steps {
                        sh '''
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Stage HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }

        stage('Prod E2E') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'https://startling-pie-e49a28.netlify.app'
            }

            steps {
                sh '''
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Prod HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

    }
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
}