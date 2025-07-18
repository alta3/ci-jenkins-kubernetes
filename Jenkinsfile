pipeline {
    agent any
    environment {
        REGISTRY = 'localhost:5000'
        IMAGE_NAME = 'simple-app'
        IMAGE_TAG = 'v1'
    }
    stages {
        stage('Kaniko Smoke Test') {
            agent {
                kubernetes {
                    label 'kaniko'
                    inheritFrom 'kaniko'
                    defaultContainer 'kaniko'
                }
            }
            environment {
                DOCKER_CONFIG = '/kaniko/.docker/'
            }
            steps {
                sh '''
                    /kaniko/executor \
                      --context . \
                      --dockerfile Dockerfile \
                      --destination=$REGISTRY/$IMAGE_NAME:$IMAGE_TAG \
                      --insecure \
                      --skip-tls-verify \
                      --verbosity debug
                '''
            }
        }
    }
}
