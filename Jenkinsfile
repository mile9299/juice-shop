pipeline {
    agent any
    
    environment {
        JUICE_SHOP_REPO = 'https://github.com/bkimminich/juice-shop.git'
        NODEJS_VERSION = '21.6.1' // Adjust the Node.js version as needed
    }
    // added 
    stages {
        stage('Install Node.js') {
            steps {
                script {
                    def nodejsTool = tool name: "NodeJS", type: 'jenkins.plugins.nodejs.tools.NodeJSInstallation'
                    if (nodejsTool) {
                        env.PATH = "${nodejsTool}/bin:${env.PATH}"
                    } else {
                        error "NodeJS ${NODEJS_VERSION} not found. Please configure it in Jenkins Global Tool Configuration."
                    }
                }
            }
        }

        stage('Checkout') {
            steps {
                script {
                    // Checkout the Juice Shop repository
                    checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: JUICE_SHOP_REPO]]])
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
        stage('Build') {
            steps {
                script {
                    // Assuming your build process, for example, using npm
                    sh 'npm cache clean -f'
                    sh 'npm install'
                    sh 'npm start'
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
                    sh "DOCKER_HOST_PORT=\$(docker port $DOCKER_PORT 3000 | cut -d ':' -f 2)"

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
