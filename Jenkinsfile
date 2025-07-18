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
  hostAliases:
    - ip: 10.7.10.47
      hostnames:
        - "bchd.registry"
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
      image: bitnami/kubectl:latest
      command:
        - /bin/sh
        - -c
        - sleep 99d
      tty: true
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
    IMAGE = "bchd.registry:2345/myapp:${env.BUILD_NUMBER}"
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
              --context \"${WORKSPACE}\" \
              --dockerfile Dockerfile \
              --destination=\"${IMAGE}\" \
              --insecure \
              --skip-tls-verify
          """
        }
      }
    }
    stage('Deploy to Kubernetes') {
      steps {
        container('kubectl') {
          sh '''
            kubectl set image -f k8s/deployment.yaml myapp=${IMAGE}
            kubectl apply -f k8s/service.yaml
          '''
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
