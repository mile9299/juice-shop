pipeline {
    agent any
    
    environment {
        JUICE_SHOP_REPO = 'https://github.com/mile9299/juice-shop.git'
        DOCKER_PORT = 3000 // Default Docker port
    }
    
    tools {
        nodejs 'NodeJS 20.0.0' // Change to a different version of Node.js
        snyk 'snyk-manual' // Ensure 'snyk_latest' matches the name of the Snyk tool configured in Jenkins
    }

    stages {
        stage('Preparation') {
            steps {
                script {
                    def nodeVersion = sh(script: "node -v", returnStdout: true).trim()
                    echo "Current Node.js version: ${nodeVersion}"
                    if (!nodeVersion.contains('20.0.0')) {
                        error "Incorrect Node.js version: ${nodeVersion}. Expected: v18.0.0"
                    }
                }
            }
        }
        
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: JUICE_SHOP_REPO]]])
            }
        }
        
        stage('Test with Snyk') {
            steps {
                snykSecurity failOnIssues: false, severity: 'critical', snykInstallation: 'snyk-manual', snykTokenId: 'SNYK'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install --legacy-peer-deps'
            }
        }

        stage('Build Frontend') {
            steps {
                sh 'cd frontend && npm install --legacy-peer-deps && npm run build:frontend'
            }
        }

        stage('Build Backend') {
            steps {
                sh 'npm run build:server'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker stop juice-shop || true
                docker rm juice-shop || true
                docker build -t juice-shop .
                '''
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def containerId = sh(script: "docker run -d -P --name juice-shop juice-shop", returnStdout: true).trim()
                    def dockerHostPort = sh(script: "docker port ${containerId} ${DOCKER_PORT} | cut -d ':' -f 2", returnStdout: true).trim()
                    echo "Juice Shop is running on http://localhost:${dockerHostPort}"
                }
            }
        }
    }
    
    post {
        success {
            echo 'Build, test, and deployment successful!'
        }
        failure {
            echo 'Build, test, or deployment failed!'
        }
    }
}
