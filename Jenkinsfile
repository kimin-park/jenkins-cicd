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
                  sudo docker build -t lkasd7512/nginx-proxy:1.2 .
                  sudo docker push lkasd7512/nginx-proxy:1.2
                  '''
                }
              }
          }
        stage('Authenticate with Azure') {
            steps {
                withEnv(['AZURE_SUBSCRIPTION_ID=3d77ae0d-8ae2-43fe-920e-af78b3ab5519', 'AZURE_TENANT_ID=c81613f1-8384-4d3d-9416-1a0259a03d57'])
                def resourceGroup = 'project2-msa-rg'
                def aksName = 'msacluster'
                withCredentials([usernamePassword(credentialsId: 'azure-jenkins', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
                withAzureCliInstallation('default') {
                azureLogin(credentialsId: 'azure-jenkins', servicePrincipal: true, tenantId: 'c81613f1-8384-4d3d-9416-1a0259a03d57')
              }
           }
        }
        stage('deploy kubernetes') {
            steps {
                withAzureCliInstallation('default') {
                    sh '''
                    az aks get-credentials --resource-group project2-msa-rg --name msacluster
                    kubectl create deployment nginx-proxy --image=lkasd7512/nginx-proxy:1.0
                    kubectl expose deployment nginx-proxy --type=LoadBalancer --port=80 --target-port=80 --name=nginx-proxy
                    '''
                }
            }
        }
    }
}
}