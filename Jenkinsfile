pipeline {
    agent any

    environment {
        NAMESPACE = "my-app"
    }

    stages {

        // ==============================
        // 1. FULL RESET
        // ==============================
        stage('Clean Kubernetes (Full Reset)') {
            steps {
                sh '''
                echo "=== Delete Existing Namespace ==="
                kubectl delete namespace ${NAMESPACE} --ignore-not-found=true
                sleep 8
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

        // ==============================
        // 2. CREATE SECRET (No YAML file)
        // ==============================
        stage('Create Secret from Jenkins') {
            steps {
                sh '''
                echo "=== Create .env File ==="

                cat <<EOF > .env
POSTGRES_DB=mydb
POSTGRES_USER=kogo
POSTGRES_PASSWORD=math1106
DBNAME=mydb
DBUSER=kogo
DBPASSWORD=math1106
DBHOST=db
DBPORT=5432
DJANGO_SECRET_KEY=django-secret
DJANGO_DEBUG=False
DJANGO_ALLOWED_HOSTS=localhost,127.0.0.1
CORS_ALLOWED_ORIGINS=http://localhost
NEXT_PUBLIC_API_BASE_URL=/api
EOF

                echo "=== Create Kubernetes Secret ==="
                kubectl create secret generic postgres-secret \
                    --from-env-file=.env \
                    -n ${NAMESPACE}
                '''
            }
        }

        // ==============================
        // 3. BUILD IMAGES
        // ==============================
        stage('Build Docker Images') {
            steps {
                sh '''
                echo "=== Build Backend ==="
                docker build --no-cache -t backend:latest ./backend

                echo "=== Build Frontend ==="
                docker build --no-cache -t frontend:latest ./frontend

                echo "=== Build Nginx ==="
                docker build --no-cache -t nginx-local:latest ./nginx
                '''
            }
        }

        // ==============================
        // 4. DEPLOY
        // ==============================
        stage('Deploy Fresh to Kubernetes') {
            steps {
                sh '''
                echo "=== Apply Kubernetes Manifests ==="
                kubectl apply -R -f k8s/
                '''
            }
        }

        // ==============================
        // 5. WAIT FOR READY
        // ==============================
        stage('Wait for Rollout') {
            steps {
                sh '''
                echo "=== Waiting for Deployments ==="

                kubectl rollout status deployment/backend -n ${NAMESPACE}
                kubectl rollout status deployment/frontend -n ${NAMESPACE}
                kubectl rollout status deployment/nginx -n ${NAMESPACE}
                '''
            }
        }

        // ==============================
        // 6. VERIFY
        // ==============================
        stage('Verify Deployment') {
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
