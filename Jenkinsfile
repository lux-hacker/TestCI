pipeline {
    agent any
    environment {
        REGISTRY = 'ghcr.io'
        TAG = 'dev'
    }
    parameters {
        choice(name: 'BRANCH', choices: ['main', 'dev'], description: 'Branch to build')
    }
    stages {
        stage('Initialize') {
            steps {
                script {
                    // Извлекаем владельца репозитория из URL
                    def repoUrl = scm.userRemoteConfigs[0].url
                    def matcher = (repoUrl =~ /github\.com[:\/]([^\/]+)\/([^\/.]+)/)
                    if (matcher) {
                        env.IMAGE_NAME = "${matcher[0][1]}/testci"
                    } else {
                        error "Failed to parse repository owner from URL: ${repoUrl}"
                    }
                }
            }
        }
        
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Login to GHCR') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-docker-creds',
                    usernameVariable: 'GITHUB_USER',
                    passwordVariable: 'GITHUB_TOKEN'
                )]) {
                    sh '''
                        docker login ${REGISTRY} \
                        -u ${GITHUB_USER} \
                        -p ${GITHUB_TOKEN}
                    '''
                }
            }
        }

        stage('Setup Buildx') {
            steps {
                sh '''
                    docker buildx create --use --name mybuilder \
                    --driver docker-container \
                    --driver-opt network=host
                    docker buildx inspect --bootstrap
                '''
            }
        }

        stage('Build and Push') {
            steps {
                sh """
                    docker buildx build \
                    --push \
                    --cache-from type=registry,ref=${REGISTRY}/${IMAGE_NAME}:latest \
                    --cache-to type=inline \
                    -t ${REGISTRY}/${IMAGE_NAME}:latest \
                    -t ${REGISTRY}/${IMAGE_NAME}:${TAG} \
                    -f ./Dockerfile \
                    .
                """
            }
        }
    }
    
    post {
        always {
            sh 'docker buildx rm mybuilder || true'
        }
    }
}