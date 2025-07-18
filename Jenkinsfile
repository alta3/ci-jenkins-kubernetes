pipeline {
  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins
  volumes:
    - name: kaniko-secret
      projected:
        sources:
          - secret:
              name: reg-credentials
              items:
                - key: .dockerconfigjson
                  path: config.json
  containers:
    - name: kaniko
      image: gcr.io/kaniko-project/executor:latest
      command:
        - /busybox/cat
      tty: true
      volumeMounts:
        - name: kaniko-secret
          mountPath: /kaniko/.docker
    - name: kubectl
      image: bitnami/kubectl:latest
      command:
        - /bin/sh
        - -c
        - sleep 99d
      tty: true
  restartPolicy: Never
"""
    }
  }
  environment {
    IMAGE = "bchd.registry/myapp:${env.BUILD_NUMBER}"
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    stage('Run Tests') {
      steps {
        container('kaniko') {
          sh '''
            python3 -m venv venv
            . venv/bin/activate
            pip install -r requirements.txt
            python3 -m pytest test_app.py
          '''
        }
      }
    }
    stage('Build & Push Image') {
      steps {
        container('kaniko') {
          sh '''
            /kaniko/executor \
              --context `pwd` \
              --dockerfile Dockerfile \
              --destination="${IMAGE}"
          '''
        }
      }
    }
    stage('Deploy to Kubernetes') {
      steps {
        container('kubectl') {
          withCredentials([file(credentialsId: 'kubeconfig-id', variable: 'KUBECONFIG')]) {
            sh '''
              echo "$KUBECONFIG" > /tmp/kubeconfig
              kubectl --kubeconfig=/tmp/kubeconfig set image -f k8s/deployment.yaml myapp="${IMAGE}"
              kubectl --kubeconfig=/tmp/kubeconfig apply -f k8s/service.yaml
            '''
          }
        }
      }
    }
  }
  post {
    success {
      echo "Deployed ${IMAGE} successfully 🎉"
    }
    failure {
      echo "Build or deploy FAILED"
    }
  }
}
