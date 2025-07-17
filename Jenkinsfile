pipeline {
    agent any
    environment {
        REGISTRY = 'localhost:5000'
        IMAGE_NAME = 'simple-app'
        IMAGE_TAG = 'v1'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Unit Tests') {
            steps {
                sh 'pip install -r requirements.txt'
                sh 'pytest test_app.py'
            }
        }
        stage('Build Image') {
            steps {
                sh 'docker build -t $REGISTRY/$IMAGE_NAME:$IMAGE_TAG .'
            }
        }
        stage('Push Image') {
            steps {
                sh 'docker push $REGISTRY/$IMAGE_NAME:$IMAGE_TAG'
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                cat <<EOF | kubectl apply -f -
                apiVersion: apps/v1
                kind: Deployment
                metadata:
                  name: simple-app
                spec:
                  replicas: 1
                  selector:
                    matchLabels:
                      app: simple-app
                  template:
                    metadata:
                      labels:
                        app: simple-app
                    spec:
                      containers:
                      - name: app
                        image: $REGISTRY/$IMAGE_NAME:$IMAGE_TAG
                        ports:
                        - containerPort: 5000
                ---
                apiVersion: v1
                kind: Service
                metadata:
                  name: simple-app
                spec:
                  type: NodePort
                  selector:
                    app: simple-app
                  ports:
                  - port: 5000
                    targetPort: 5000
                    nodePort: 31234
                EOF
                '''
            }
        }
        stage('Integration Test') {
            steps {
                sh 'curl -s http://localhost:31234 | grep "Hello, Kubernetes!"'
            }
        }
    }
}
