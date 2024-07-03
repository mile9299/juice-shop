pipeline {
    agent any

    environment {
        JUICE_SHOP_REPO = 'https://github.com/mile9299/juice-shop.git'
        DOCKER_PORT = 3000 // Default Docker port
    }
    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: JUICE_SHOP_REPO]]])
                }
            }
        }
        
        stage('Install Node.js') {
            steps {
                sh 'curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash' // Install nvm
                sh '. ~/.nvm/nvm.sh && nvm install 18.16.1' // Install Node.js version 18.16.1
                sh '. ~/.nvm/nvm.sh && nvm use 18.16.1' // Use Node.js version 18.16.1
                sh '. ~/.nvm/nvm.sh && node -v' // Verify the Node.js version
                sh '. ~/.nvm/nvm.sh && npm -v' // Verify npm version
            }
        }

        stage('Build') {
            steps {
                script {
                    sh '. ~/.nvm/nvm.sh && npm cache clean -f'
                    sh '. ~/.nvm/nvm.sh && npm install'
                    sh '. ~/.nvm/nvm.sh && npm install --force'
                    // Start the application in the background using nohup
                    sh '. ~/.nvm/nvm.sh && nohup npm start > /dev/null 2>&1 &'

                    // Sleep for a few seconds to ensure the application has started before moving to the next stage
                    sleep(time: 5, unit: 'SECONDS')
                }
            }
        }
        
        stage('Test with Snyk') {
            steps {
                script {
                    snykSecurity failOnIssues: false, severity: 'critical', snykInstallation: 'snyk-manual', snykTokenId: 'SNYK'
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    // Stop and remove the container if it exists
                    sh 'docker stop juice-shop || true'
                    sh 'docker rm juice-shop || true'
                    // Build and run the Docker container with a dynamically allocated port
                    sh "docker build -t juice-shop ."
                    sh "DOCKER_PORT=\$(docker run -d -P --name juice-shop juice-shop)"
                    sh "DOCKER_HOST_PORT=\$(docker port \$DOCKER_PORT 3000 | cut -d ':' -f 2)"
                    echo "Juice Shop is running on http://localhost:\$DOCKER_HOST_PORT"
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
