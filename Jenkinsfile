pipeline {
    agent any

    parameters {
        booleanParam(name: 'BUILD_DOCKER', defaultValue: true, description: 'Build Docker Image')
        booleanParam(name: 'DOCKER_DEPLOY', defaultValue: true, description: 'Docker Deploy')
        choice(name: "TEST_CHOICE", choices: ["production", "staging"], description: "Sample multi-choice parameter")
        string(name: 'REGISTRY_DOCKER', defaultValue: 'muyleangin', description: 'Registry')
        string(name: 'BUILD_CONTAINER_NAME', defaultValue: 'alumni-ui', description: 'Container')
        string(name: 'CONTAINER_NAME', defaultValue: 'alumni-ui', description: 'Container')
        string(name: 'DOCKER_TAG', defaultValue: 'latest', description: 'Docker Tag')
        string(name: 'REPO_URL', defaultValue: 'http://git.istad.co/cstad-ite-2nd-generation/fswd/alumni/alumni-ui.git', description: 'Repository URL')
        string(name: 'DEPLOY_PORT', defaultValue: '3106', description: 'Port for deployment')
        string(name: 'COMPOSE_FILE', defaultValue: 'docker-compose.yml', description: 'Docker Compose file')
    }

    environment {
        TELEGRAM_BOT_TOKEN = '7390043101:AAEMaazFdCsEp77Q0GiqjBSwCKGReVBepyI'
        TELEGRAM_CHAT_ID = '-4133149806'
        CREDENTIAL_GIT = 'gitlab-token'
        BRANCH = 'main'
    }

    stages {
        stage('Get Code from SCM') {
            steps {
                echo "TEST_CHOICE is ${params.TEST_CHOICE}"
                script {
                    git branch: "${BRANCH}", url: "${params.REPO_URL}", credentialsId: "${CREDENTIAL_GIT}"
                }
            }
        }
        stage('Build Docker Image') {
            when {
                expression { return params.BUILD_DOCKER }
            }
            steps {
                script {
                    def dockerImage = "${params.REGISTRY_DOCKER}/${params.BUILD_CONTAINER_NAME}:${params.DOCKER_TAG}"

                    // Check if the Docker image already exists and remove it
                    def imageExists = sh(script: "docker images -q ${dockerImage}", returnStdout: true).trim()
                    if (imageExists) {
                        sh "docker rmi -f ${dockerImage}"
                    }

                    // Dockerfile content
                    def dockerfileContent = '''
                        FROM node:18-alpine AS builder

                        RUN apk add --no-cache libc6-compat

                        WORKDIR /app

                        COPY package.json package-lock.json ./
                        COPY . .

                        RUN npm i sharp
                        RUN npm run build

                        FROM node:18-alpine

                        RUN apk update && apk upgrade && apk add dumb-init && adduser -D nextuser

                        WORKDIR /app

                        COPY --chown=nextuser:nextuser --from=builder /app/public ./public
                        COPY --chown=nextuser:nextuser --from=builder /app/.next/standalone ./
                        COPY --chown=nextuser:nextuser --from=builder /app/.next/static ./.next/static

                        USER nextuser

                        EXPOSE 3000

                        ENV HOST=0.0.0.0 PORT=3000 NODE_ENV=production

                        CMD ["dumb-init", "node", "server.js"]
                    '''
                    writeFile file: 'Dockerfile', text: dockerfileContent

                    // Build the Docker image
                    sh "docker build -t ${dockerImage} -f Dockerfile ."
                }
            }
        }
        stage('Push Docker Image to Registry') {
            when {
                expression { return params.BUILD_DOCKER }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                    script {
                        def dockerImage = "${params.REGISTRY_DOCKER}/${params.BUILD_CONTAINER_NAME}:${params.DOCKER_TAG}"
                        sh "docker login -u ${USERNAME} -p ${PASSWORD}"
                        sh "docker push ${dockerImage}"
                    }
                }
            }
        }
        stage('Deploy Docker') {
            when {
                expression { return params.DOCKER_DEPLOY }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                    script {
                        def dockerImage = "${params.REGISTRY_DOCKER}/${params.BUILD_CONTAINER_NAME}:${params.DOCKER_TAG}"

                        // Check if the Docker container is running and stop it if necessary
                        def containerExists = sh(script: "docker ps -q --filter 'name=${params.CONTAINER_NAME}'", returnStdout: true).trim()
                        if (containerExists) {
                            sh "docker stop ${params.CONTAINER_NAME}"
                            sh "docker rm ${params.CONTAINER_NAME}"
                        }

                        // Check if the Docker image already exists and remove it if necessary
                        def imageExists = sh(script: "docker images -q ${dockerImage}", returnStdout: true).trim()
                        if (imageExists) {
                            sh "docker rmi -f ${dockerImage}"
                        }

                        // Log in to Docker registry
                        sh "docker login -u ${USERNAME} -p ${PASSWORD}"

                        def composeContent = """
                            version: '3'
                            services:
                              ${params.BUILD_CONTAINER_NAME}:
                                image: ${dockerImage}
                                ports:
                                  - "${params.DEPLOY_PORT}:3000"
                                container_name: ${params.CONTAINER_NAME}
                                networks:
                                  - data_analytics
                            networks:
                              data_analytics:
                                external: true
                        """
                        writeFile file: "${params.COMPOSE_FILE}", text: composeContent

                        // Remove existing Docker Compose services if any
                        sh """
                            if [ -f ${params.COMPOSE_FILE} ]; then
                                docker-compose -f ${params.COMPOSE_FILE} down
                            fi
                        """

                        // Deploy with Docker Compose
                        sh "docker-compose -f ${params.COMPOSE_FILE} up -d"

                        // Get the public IP address and port for deployment
                        def ipAddress = sh(script: 'curl -s ifconfig.me', returnStdout: true).trim()
                        def ipWithPort = "${ipAddress}:${params.DEPLOY_PORT}"
                        env.DEPLOY_IP_PORT = ipWithPort
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                def successMessage = """
                    *Deployment Successful:*
                    - *Container:* ${params.CONTAINER_NAME}
                    - *Branch:* ${BRANCH}
                    - *Deployment URL:* [${env.DEPLOY_IP_PORT}](http://${env.DEPLOY_IP_PORT})
                """
                sendTelegramMessage(successMessage)
            }
        }
        failure {
            script {
                sendTelegramMessage("Deployment Failed: ${params.CONTAINER_NAME} on branch ${BRANCH}")
            }
        }
    }
}

def sendTelegramMessage(message) {
    sh "curl -s -X POST https://api.telegram.org/bot${env.TELEGRAM_BOT_TOKEN}/sendMessage -d chat_id=${env.TELEGRAM_CHAT_ID} -d parse_mode=Markdown -d text='${message}'"
}

