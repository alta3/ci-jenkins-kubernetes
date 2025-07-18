pipeline {
    agent any
    stages {
        stage('Kaniko Smoke Test') {
            agent {
                kubernetes {
                    label 'kaniko'
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
                      --destination=localhost:5000/test-image:v1 \
                      --insecure \
                      --skip-tls-verify \
                      --verbosity debug || true
                '''
            }
        }
    }
}
