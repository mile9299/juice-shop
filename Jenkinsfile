pipeline {
    agent {
        docker {
        //image 'docker:latest'
        image 'node:14'
        args '--privileged -v /var/run/docker.sock:/var/run/docker.sock'
        }
         
    }

    environment {
        JUICE_SHOP_REPO = 'https://github.com/bkimminich/juice-shop.git'
        DOCKER_PORT = 3000 // Default Docker port
        SPECTRAL_DSN = credentials('SPECTRAL_DSN')
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: JUICE_SHOP_REPO]]])
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    docker.image('node:14').inside {
                        sh '/usr/share/nodejs/npm/bin/npm cache clean -f'
                        sh '/usr/share/nodejs/npm/bin/npm install --force'
                    // Start the application in the background using nohup
                        //sh '/usr/share/nodejs/npm/bin/npm start > /dev/null 2>&1 &'
                    // Start the application in the background using nohup
                        sh 'nohup /usr/share/nodejs/npm/bin/npm start > /dev/null 2>&1 &'
                    // Sleep for a few seconds to ensure the application has started before moving to the next stage
                        sleep(time: 5, unit: 'SECONDS')
                       }
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

        stage('install Spectral') {
            steps {
                sh "curl -L 'https://get.spectralops.io/latest/x/sh?dsn=$SPECTRAL_DSN' | sh"
            }
        }

        stage('scan for issues') {
            steps {
                sh "$HOME/.spectral/spectral scan --ok --engines secrets,iac,oss --include-tags base,audit,iac"
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
