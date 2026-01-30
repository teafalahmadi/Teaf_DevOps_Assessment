pipeline {
  agent none

  environment {
    DOCKERHUB = credentials('dockerhub-creds')  // Username/Password credential
    IMAGE_NAME = "${DOCKERHUB_USR}/simple-java-app"
    IMAGE_TAG  = "${BUILD_NUMBER}"
  }

  stages {

    stage('Build & Test (Maven)') {
      agent {
        kubernetes {
          yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.9-eclipse-temurin-17
    command: ['cat']
    tty: true
"""
        }
      }
       steps {
          checkout scm
          dir('simple-java-app') {
            container('maven') {
              sh 'ls -la'
              sh 'mvn -B clean test package'
      }
    }
  }
}

    stage('Build & Push Image (Kaniko)') {
      agent {
        kubernetes {
          yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    command: ['cat']
    tty: true
    volumeMounts:
    - name: docker-config
      mountPath: /kaniko/.docker
  volumes:
  - name: docker-config
    secret:
      secretName: dockerhub-config
"""
        }
      }
      steps {
        checkout scm
        container('kaniko') {
          sh 'ls -la'
          sh 'test -f Dockerfile'
          sh """
            /kaniko/executor \
              --context $WORKSPACE \
              --dockerfile $WORKSPACE/Dockerfile \
              --destination ${IMAGE_NAME}:${IMAGE_TAG} \
              --destination ${IMAGE_NAME}:latest
          """
        }
      }
    }

    stage('Deploy to Kubernetes') {
      agent {
        kubernetes {
          yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kubectl
    image: bitnami/kubectl:latest
    command: ['cat']
    tty: true
"""
        }
      }
      steps {
        checkout scm
        container('kubectl') {
          sh "sed -i 's|image: .*simple-java-app.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|' k8s/deployment.yaml"
          sh 'kubectl apply -f k8s/'
          sh 'kubectl rollout status deployment/simple-java-app --timeout=180s'
        }
      }
    }
  }
}
