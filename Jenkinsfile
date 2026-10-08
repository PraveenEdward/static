pipeline {

    parameters {
        string(
            name: 'IMAGE_VERSION',
            defaultValue: 'v0',
            description: 'Docker image version to build and deploy'
        )

        choice(
            name: 'AGENT',
            choices: [
                'built-in'
            ],
            description: 'Select Jenkins agent'
        )
    }

    agent {
        label "${params.AGENT}"
    }

    environment {
        IMAGE = "praveenedward/static"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Image') {
            steps {
                sh """
                    docker build -t ${IMAGE}:${IMAGE_VERSION} ./app
                """
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
                sh """
                    docker push ${IMAGE}:${IMAGE_VERSION}
                """
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
                sh """
                    kubectl set image deployment/static-deployment \
                        static=${IMAGE}:${IMAGE_VERSION}
                """
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
            echo "Deployment successful: ${IMAGE}:${IMAGE_VERSION}"
        }

        failure {
            echo "Deployment failed"
        }
    }
}
