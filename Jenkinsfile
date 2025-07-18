pipeline {
  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins: kaniko-deploy
spec:
  serviceAccountName: jenkins
  containers:
    - name: kaniko
      image: gcr.io/kaniko-project/executor:debug
      command:
        - /busybox/cat
      tty: true
      volumeMounts:
        - name: kaniko-secret
          mountPath: /kaniko/.docker
    - name: kubectl
      image: bitnami/kubectl:1.29.2
      command:
        - /bin/sh
        - -c
        - sleep 99d
      tty: true
      securityContext:
        runAsUser: 0
        runAsGroup: 0
      volumeMounts:
        - name: kubeconfig
          mountPath: /root/.kube
    - name: python
      image: python:3.12-alpine
      command:
        - /bin/sh
        - -c
        - sleep 99d
      tty: true
  volumes:
    - name: kaniko-secret
      projected:
        sources:
          - secret:
              name: reg-credentials
              items:
                - key: .dockerconfigjson
                  path: config.json
    - name: kubeconfig
      projected:
        sources:
          - secret:
              name: kubeconfig-id
              items:
                - key: config
                  path: config
  restartPolicy: Never
"""
    }
  }
  environment {
      IMAGE = "ghcr.io/csfeeser/ci-jenkins-kubernetes/myapp:${BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    stage('Run Tests') {
      steps {
        container('python') {
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
          sh """
            /kaniko/executor \
              --context "${WORKSPACE}" \
              --dockerfile Dockerfile \
              --destination="${IMAGE}" \
              --skip-tls-verify-pull
          """
        }
      }
    }
    stage('Deploy to Kubernetes') {
      steps {
        container('kubectl') {
          script {
            sh "sed -i 's|placeholder|${env.IMAGE}|' k8s/deployment.yaml"
            sh 'kubectl apply -f k8s/deployment.yaml'
            sh 'kubectl apply -f k8s/service.yaml'
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
