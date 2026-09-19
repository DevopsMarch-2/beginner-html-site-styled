pipeline {
    agent { label 'k8s-master' } // Runs build steps on K8s Master agent
    
    environment {
        DOCKER_HUB = 'YOUR_DOCKERHUB_USERNAME' // Replace with your actual Docker Hub username
        IMAGE_NAME = 'beginner-html-site'
        REGISTRY_CRED = 'dockerhub-credentials-id' // Jenkins Credential ID for Docker Hub
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/DevopsMarch-2/beginner-html-site-styled.git'
            }
        }
        
        stage('Build & Push Docker Image') {
            steps {
                script {
                    // Log in to Docker Hub using stored Jenkins credentials
                    withCredentials([usernamePassword(credentialsId: "${REGISTRY_CRED}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        
                        // Build Docker image
                        sh "docker build -t ${DOCKER_HUB}/${IMAGE_NAME}:${BUILD_NUMBER} ."
                        sh "docker tag ${DOCKER_HUB}/${IMAGE_NAME}:${BUILD_NUMBER} ${DOCKER_HUB}/${IMAGE_NAME}:latest"
                        
                        // Push Docker image to Docker Hub
                        sh "docker push ${DOCKER_HUB}/${IMAGE_NAME}:${BUILD_NUMBER}"
                        sh "docker push ${DOCKER_HUB}/${IMAGE_NAME}:latest"
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    sed -i "s|DOCKERHUB_USERNAME|${DOCKER_HUB}|g" deployment.yaml
                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml
                    kubectl rollout restart deployment/html-site-deployment
                '''
            }
        }
    }
    
    post {
        always {
            // Clean up Docker login credentials after build finishes
            sh 'docker logout'
        }
    }
}
