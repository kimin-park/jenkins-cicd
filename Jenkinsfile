pipeline {
  agent any
  
  stages {
    stage('Build Docker Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'p2ace0fm1nd!', usernameVariable: 'lkasd7512')]) {
          script {
            docker.withRegistry("${DOCKER_REGISTRY}", 'docker-hub-credentials') {
              def dockerImage = docker.build("${DOCKER_REPOSITORY}/nginx-proxy:${env.BUILD_NUMBER}")
              dockerImage.push()
            }
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
