pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = "mathkogogany1490"   // 🔥 실제 DockerHub ID
        DOCKER_CREDENTIALS = "dockerhub-creds"
        KUBE_NAMESPACE = "my-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Backend Image') {
            steps {
                script {
                    docker.build("${DOCKERHUB_REPO}/backend:${IMAGE_TAG}", "./backend")
                    docker.build("${DOCKERHUB_REPO}/backend:latest", "./backend")
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                script {
                    docker.build(
                        "${DOCKERHUB_REPO}/frontend:${IMAGE_TAG}",
                        "--build-arg NEXT_PUBLIC_API_BASE_URL=/api ./frontend"
                    )
                    docker.build(
                        "${DOCKERHUB_REPO}/frontend:latest",
                        "--build-arg NEXT_PUBLIC_API_BASE_URL=/api ./frontend"
                    )
                }
            }
        }

        stage('Build Nginx Image') {
            steps {
                script {
                    docker.build("${DOCKERHUB_REPO}/nginx:${IMAGE_TAG}", "./nginx")
                    docker.build("${DOCKERHUB_REPO}/nginx:latest", "./nginx")
                }
            }
        }

        stage('Push Images') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_CREDENTIALS) {  // 🔥 URL 제거

                        docker.image("${DOCKERHUB_REPO}/backend:${IMAGE_TAG}").push()
                        docker.image("${DOCKERHUB_REPO}/backend:latest").push()

                        docker.image("${DOCKERHUB_REPO}/frontend:${IMAGE_TAG}").push()
                        docker.image("${DOCKERHUB_REPO}/frontend:latest").push()

                        docker.image("${DOCKERHUB_REPO}/nginx:${IMAGE_TAG}").push()
                        docker.image("${DOCKERHUB_REPO}/nginx:latest").push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                kubectl apply -f k8s/

                kubectl set image deployment/backend backend=${DOCKERHUB_REPO}/backend:${IMAGE_TAG} -n ${KUBE_NAMESPACE}
                kubectl set image deployment/frontend frontend=${DOCKERHUB_REPO}/frontend:${IMAGE_TAG} -n ${KUBE_NAMESPACE}
                kubectl set image deployment/nginx nginx=${DOCKERHUB_REPO}/nginx:${IMAGE_TAG} -n ${KUBE_NAMESPACE}

                kubectl rollout status deployment/backend -n ${KUBE_NAMESPACE}
                kubectl rollout status deployment/frontend -n ${KUBE_NAMESPACE}
                kubectl rollout status deployment/nginx -n ${KUBE_NAMESPACE}
                """
            }
        }
    }

    post {
        success {
            echo "Deployment Success 🚀 Build: ${IMAGE_TAG}"
        }
        failure {
            echo "Deployment Failed ❌"
        }
    }
}
