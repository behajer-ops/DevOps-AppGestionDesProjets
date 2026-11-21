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
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig(credentialsId: 'minikube-kubeconfig') {
                    script {
                        // 1. Applique tous les manifestes YAML (Deployments, Services, Ingress)
                        sh 'kubectl apply -f k8s/'
                        
                        // 2. Force le redémarrage des pods pour prendre la nouvelle image 'latest'
                        sh 'kubectl rollout restart deployment backend-deployment'
                        sh 'kubectl rollout restart deployment frontend-deployment'
                        
                        // 3. Attend que le déploiement soit terminé
                        sh 'kubectl rollout status deployment backend-deployment --timeout=120s'
                        sh 'kubectl rollout status deployment frontend-deployment --timeout=120s'
                        
                        // 4. Affiche l'état final
                        sh 'kubectl get pods'
                        sh 'kubectl get ingress'
                    }
                }
            }
        }
    }
}
