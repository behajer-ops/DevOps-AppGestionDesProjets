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
                    sh 'NODE_OPTIONS="--max-old-space-size=4096" npm run build -- --configuration production'
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    // Analyse du Backend
                    dir('backend') {
                        withSonarQubeEnv('SonarQube') {
                            sh 'mvn sonar:sonar'
                        }
                    }
                    // Analyse du Frontend
                    dir('frontend') {
                        withSonarQubeEnv('SonarQube') {
                            // Le scanner utilise sonar-project.properties
                            sh 'npx sonar-scanner'
                        }
                    }
                }
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    docker.withRegistry('', 'docker-hub-credentials') {
                        def backendImage = docker.build("behajer/backend-devops:${env.BUILD_NUMBER}", "./backend")
                        backendImage.push()
                        backendImage.push('latest')
                    }
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
