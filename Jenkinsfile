pipeline {
    agent { label 'k8s-master' }
    environment {
        DOCKER_HUB = 'venkateshhosamani'
        IMAGE_NAME = 'beginner-html-site'
        REGISTRY_CRED = 'dockerhub-credentials-id'
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
                    docker.withRegistry('', REGISTRY_CRED) {
                        def customImage = docker.build("${DOCKER_HUB}/${IMAGE_NAME}:${BUILD_NUMBER}")
                        customImage.push()
                        customImage.push("latest")
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
}
