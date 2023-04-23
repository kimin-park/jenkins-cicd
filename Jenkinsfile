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
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                  sh '''
                  sudo docker login -u $DOCKER_USER -p $DOCKER_PASS
                  sudo docker build -t lkasd7512/nginx-proxy:1.0 .
                  sudo docker push lkasd7512/nginx-proxy:1.0
                  '''
                }
              }
          }
        stage('Authenticate with Azure') {
            steps {
                withAzureCliInstallation('default') {
                azureLogin(credentialsId: 'azure-service-principal', servicePrincipal: true, tenantId: 'c81613f1-8384-4d3d-9416-1a0259a03d57')
              }
           }
        }
        stage('deploy kubernetes') {
            steps {
                withAzureCliInstallation('default') {
                    sh '''
                    az aks get-credentials --resource-group projec2-msa-rg --name msacluster
                    kubectl create deployment nginx-proxy --image=lkasd7512/nginx-proxy:1.0
                    kubectl expose deployment nginx-proxy --type=LoadBalancer --port=80 --target-port=80 --name=nginx-proxy
                    '''
                }
            }
        }
    }
}
