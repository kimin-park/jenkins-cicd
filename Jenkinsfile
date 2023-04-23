pipeline {
    agent any
    stages {
        stage('git scm update') {
          steps {
              git url: 'https://github.com/kimin-park/jenkins-cicd.git' , branch: 'main'
          }
        }
        stage('docker build and push') {
            steps {
              sh '''
              docker build -t lkasd7512/nginx-proxy:1.0 .
              docker push lkasd7512/nginx-proxy:1.0
              '''
                
            }
        }
        stage('deploy kubernetes') {
            steps {
                withAzureCLI([azureSubscription(credentialsId: '', subscriptionId: '')]) {
                    sh '''
                    az aks get-credentials --resource-group projec2-msa-cicd --name msacluster
                    kubectl create deployment nginx-proxy --image=lkasd7512/nginx-proxy:1.0
                    kubectl expose deployment nginx-proxy --type=LoadBalancer --port=80 --target-port=80 --name=nginx-proxy
                    '''
                }
            }
        }
    }
}
