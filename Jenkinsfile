pipeline {
    agent any
    stages {
        stage('Git clone') {
            steps {
                git branch: 'main',
                    credentialsId: 'GIT-HUB-CREDENTIALS',
                    url: 'https://github.com/behajer-ops/DevOps-AppGestionDesProjets.git'
            }
        }
        stage('Build backend') {
            steps {
                dir('backend') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }
        stage('Build frontend') {
            steps {
                dir('frontend') {
                    sh 'npm ci'
                    // Ajout de la limite de mémoire pour éviter le crash de Node
                    sh 'NODE_OPTIONS="--max-old-space-size=4096" npm run build -- --configuration production'
                }
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    // --- Backend ---
                    docker.withRegistry('', 'docker-hub-credentials') {
                        def backendImage = docker.build("behajer/backend-devops:${env.BUILD_NUMBER}", "./backend")
                        backendImage.push()
                        backendImage.push('latest')
                    }
                    // --- Frontend ---
                    docker.withRegistry('', 'docker-hub-credentials') {
                        def frontendImage = docker.build("behajer/frontend-devops:${env.BUILD_NUMBER}", "./frontend")
                        frontendImage.push()
                        frontendImage.push('latest')
                    }
                }
            }
        }
    }
}
