pipeline {
    agent any

    environment {
        NODE_VERSION = '14.x'  // Specify your Node.js version here
    }

    tools {
        nodejs "${NODE_VERSION}"  // Use Node.js version
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
                sh 'npm run test -- --watchAll=false'  // Disable interactive watch mode
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
                // Example of deployment step (adjust as needed):
                // sh 'scp -r build/* user@your-server:/var/www/html'
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }

        success {
            echo 'Build and tests passed successfully!'
        }

        failure {
            echo 'Build or tests failed!'
        }
    }
}
