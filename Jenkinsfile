pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'af4fec5f-157a-40a7-8cbb-05752e2aad98'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
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
                script {
                    try {
                        echo "🔧 Checking required files..."
                        sh '''
                            if [ ! -f index.html ]; then
                                echo "❌ Error: Missing index.html" >&2
                                exit 1
                            fi
                            if [ ! -f netlify/functions/quote.js ]; then
                                echo "❌ Error: Missing quote function file" >&2
                                exit 1
                            fi
                            echo "✅ Build check passed."
                        '''
                    } catch (Exception e) {
                        error "Build stage failed: ${e.message}"
                    }
                }
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                script {
                    try {
                        echo "🧪 Testing quote function load..."
                        sh '''
                            if node -e "require('./netlify/functions/quote.js')"; then
                                echo "✅ Function loaded successfully"
                            else
                                echo "❌ Error: Function failed to load" >&2
                                exit 1
                            fi
                        '''
                    } catch (Exception e) {
                        error "Test stage failed: ${e.message}"
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
                script {
                    try {
                        echo "🚀 Deploying to Netlify..."
                        sh '''
                            npm install netlify-cli
                            if ! node_modules/.bin/netlify deploy \
                                --auth=$NETLIFY_AUTH_TOKEN \
                                --site=$NETLIFY_SITE_ID \
                                --dir=. \
                                --prod; then
                                echo "❌ Deployment failed!" >&2
                                exit 1
                            fi
                            echo "✅ Deployment successful!"
                        '''
                    } catch (Exception e) {
                        error "Deploy stage failed: ${e.message}"
                    }
                }
            }
        }

        stage('Post Deploy') {
            steps {
                echo "✅ Deployment complete! Your app is live."
            }
        }
    }

    post {
        success {
            echo "🎉 CI/CD pipeline finished successfully."
        }
        failure {
            echo "❌ Pipeline failed. Check logs for details."
        }
    }
}
