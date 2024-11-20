pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    // Determina il tag in base al branch
                    def branch = env.GIT_BRANCH ?: 'unknown'
                    def tag = branch.contains('main') ? 'latest' : (branch.contains('develop') ? 'develop' : 'other')

                    docker.withRegistry('', DOCKERHUB_CREDENTIALS) {
                        def image = docker.build("kira002/flask-app-example:${tag}")
                        image.push()
                    }
                    // Rimuove l'immagine locale
                    sh "docker rmi --force kira002/flask-app-example:${tag}"
                }
            }
        }
    }
}
