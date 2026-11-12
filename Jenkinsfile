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
                    sh 'npm run build -- --configuration production'
                }
            }
        }
    }
}
