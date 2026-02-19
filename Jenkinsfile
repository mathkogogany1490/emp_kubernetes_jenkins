pipeline {
    agent any

    environment {
        NAMESPACE = "my-app"
    }

    stages {

        stage('Clean Kubernetes (Full Reset)') {
            steps {
                sh '''
                echo "=== Delete Existing Resources ==="
                kubectl delete namespace ${NAMESPACE} --ignore-not-found=true
                sleep 5
                '''
            }
        }

        stage('Recreate Namespace') {
            steps {
                sh '''
                echo "=== Create Namespace ==="
                kubectl create namespace ${NAMESPACE}
                '''
            }
        }

        stage('Build Images') {
            steps {
                sh '''
                echo "=== Build Docker Images ==="
                docker build --no-cache -t backend:latest ./backend
                docker build --no-cache -t frontend:latest ./frontend
                docker build --no-cache -t nginx-local:latest ./nginx
                '''
            }
        }

        stage('Deploy Fresh to Kubernetes') {
            steps {
                sh '''
                echo "=== Apply Kubernetes Manifests ==="
                kubectl apply -R -f k8s/
                '''
            }
        }

        stage('Wait for Pods') {
            steps {
                sh '''
                echo "=== Wait for Rollout ==="
                kubectl rollout status deployment/backend -n ${NAMESPACE}
                kubectl rollout status deployment/frontend -n ${NAMESPACE}
                kubectl rollout status deployment/nginx -n ${NAMESPACE}
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                echo "=== Pods Status ==="
                kubectl get pods -n ${NAMESPACE}
                echo "=== Services ==="
                kubectl get svc -n ${NAMESPACE}
                '''
            }
        }
    }
}
