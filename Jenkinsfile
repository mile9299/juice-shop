pipeline {
    agent any
    
    environment {
        JUICE_SHOP_REPO = 'https://github.com/mile9299/juice-shop.git'
        DOCKER_PORT = 3000 // Default Docker port
    }
    
    tools {
        nodejs 'NodeJS 20.0.0' // Ensure 'NodeJS 20.0.0' matches the name of the Node.js tool configured in Jenkins
    }

    stages {
        stage('Preparation') {
            steps {
                // Update npm to a specific compatible version
                sh 'npm install -g npm@9.7.0'
            }
        }
        
        stage('Checkout') {
            steps {
                // Checkout the repository
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: JUICE_SHOP_REPO]]])
            }
        }
        
        stage('Test with Snyk') {
            steps {
                // Add steps for testing with Snyk here
                // Example:
                // script {
                snykSecurity failOnIssues: false, severity: 'critical', snykInstallation: 'snyk-manual', snykTokenId: 'SNYK'
                // }
            }
        }
        
        stage('Build') {
            steps {
                // Clean npm cache, install dependencies, and start the application
                sh 'npm cache clean -f'
                sh 'npm install --force'
                sh 'nohup npm start > /dev/null 2>&1 &'
                sleep(time: 5, unit: 'SECONDS')
            }
        }
        
        stage('Deploy') {
            steps {
                // Stop and remove the container if it exists, build and run the Docker container
                sh 'docker stop juice-shop || true'
                sh 'docker rm juice-shop || true'
                sh 'docker build -t juice-shop .'
                sh "DOCKER_PORT=\$(docker run -d -P --name juice-shop juice-shop)"
                sh "DOCKER_HOST_PORT=\$(docker port $DOCKER_PORT 3000 | cut -d ':' -f 2)"
                echo "Juice Shop is running on http://localhost:\$DOCKER_HOST_PORT"
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
