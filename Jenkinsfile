pipeline {

    agent any

    environment {
        IMAGE = "praveenedward/static"
        TAG   = "v${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Image') {
            steps {
                sh '''
                    docker build -t ${IMAGE}:${TAG} ./app
                '''
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" \
                            --password-stdin
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                    docker push ${IMAGE}:${TAG}
                '''
            }
        }

        stage('Deploy Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f k8s/
                '''
            }
        }

        stage('Update Image') {
            steps {
                sh '''
                    kubectl set image deployment/static-deployment \
                        static=${IMAGE}:${TAG}
                '''
            }
        }

        stage('Rollout Status') {
            steps {
                sh '''
                    kubectl rollout status deployment/static-deployment \
                        --timeout=5m
                '''
            }
        }
    }

    post {
        success {
            echo "Deployment successful: ${IMAGE}:${TAG}"
        }

        failure {
            echo "Deployment failed"
        }
    }
}
