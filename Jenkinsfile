pipeline {
  agent any
  environment {
    DOCKER_REGISTRY = "docker.io"
    DOCKER_REPOSITORY = "yourdockerusername"
    DOCKER_USERNAME = credentials('dockerhub-username')
    DOCKER_PASSWORD = credentials('dockerhub-password')
    KUBECONFIG = credentials('aks-kubeconfig')
  }
  stages {
    stage('Build Docker Image') {
      steps {
        script {
          docker.withRegistry("${DOCKER_REGISTRY}", 'docker-hub-credentials') {
            def dockerImage = docker.build("${DOCKER_REPOSITORY}/nginx-proxy:${env.BUILD_NUMBER}")
            dockerImage.push()
          }
        }
      }
    }
    stage('Deploy to AKS') {
      steps {
        script {
          sh 'kubectl --kubeconfig=${KUBECONFIG} apply -f nginx-proxy.yaml'
        }
      }
    }
  }
}
