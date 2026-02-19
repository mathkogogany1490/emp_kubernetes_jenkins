pipeline {
    agent any

    environment {
        NAMESPACE = "my-app"
    }

    stages {

        stage('Build Images') {
            steps {
                sh '''
                docker build -t backend:latest ./backend
                docker build -t frontend:latest ./frontend
                docker build -t nginx-local:latest ./nginx
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -R -f k8s/'
            }
        }

        stage('Rollout Restart') {
            steps {
                sh '''
                kubectl rollout restart deployment backend -n ${NAMESPACE}
                kubectl rollout restart deployment frontend -n ${NAMESPACE}
                kubectl rollout restart deployment nginx -n ${NAMESPACE}
                '''
            }
        }

        stage('Wait for Rollout') {
            steps {
                sh '''
                kubectl rollout status deployment/backend -n ${NAMESPACE}
                kubectl rollout status deployment/frontend -n ${NAMESPACE}
                kubectl rollout status deployment/nginx -n ${NAMESPACE}
                '''
            }
        }

        stage('Check Pods') {
            steps {
                sh 'kubectl get pods -n ${NAMESPACE}'
            }
        }
    }
}
