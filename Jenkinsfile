pipeline {

    parameters {

        string(
            name: 'IMAGE_VERSION',
            defaultValue: 'v0',
            description: 'Docker image version to build and deploy'
        )

        choice(
            name: 'JENKINS_SERVER',
            choices: ['built-in'],
            description: 'Select Jenkins server/agent'
        )
    }

    agent {
        label "${params.JENKINS_SERVER}"
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
                    docker build -t ${IMAGE}:${params.IMAGE_VERSION} ./app
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
                    docker push ${IMAGE}:${params.IMAGE_VERSION}
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
                        static=${IMAGE}:${params.IMAGE_VERSION}
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
            echo "Deployment successful: ${IMAGE}:${params.IMAGE_VERSION}"
        }

        failure {
            echo "Deployment failed"
        }
    }
}
