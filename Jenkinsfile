pipeline {
    agent any

    environment {
        NODE_VERSION = 'NodeJS 14.x'  // Adjust this as per your Node.js version
    }

    tools {
        nodejs "${NODE_VERSION}"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Cloning repository...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing NPM dependencies...'
                sh 'npm install'
            }
        }

        stage('Build') {
            steps {
                echo 'Building the React app...'
                sh 'npm run build'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'npm run test -- --watchAll=false'
            }
        }

        stage('Lint') {
            steps {
                echo 'Linting the code...'
                sh 'npm run lint'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying the React app...'
                // Add your deployment steps here
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            // Wrapping cleanWs in node to ensure workspace context is available
            node {
                cleanWs()
            }
        }

        success {
            echo 'Build and tests passed successfully!'
        }

        failure {
            echo 'Build or tests failed!'
        }
    }
}
