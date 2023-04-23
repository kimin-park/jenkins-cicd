pipeline {
    agent any
    
    environment {
      AZURE_SERVICE_PRINCIPAL_ID = credentials('559fc471-900a-4d25-9acc-ee2b53dc4ffc')
      AZURE_SERVICE_PRINCIPAL_SECRET = credentials('P.t8Q~L8wzWNZRupbuSOADkgRvIfOVx4lrPIEdn5')
      AZURE_TENANT_ID = credentials('c81613f1-8384-4d3d-9416-1a0259a03d57')
    }
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
        stages {
          stage('Azure Authentication') {
              steps {
                script {
                    withCredentials([azureServicePrincipal('AZURE')]) {
                      def azureCredentials = azureServicePrincipal('AZURE')
                      def az = azureCLI(credentialsId: azureCredentials.id)
                      az.withAuth {
                        sh 'az login --service-principal -u $AZURE_SERVICE_PRINCIPAL_ID -p $AZURE_SERVICE_PRINCIPAL_SECRET --tenant $AZURE_TENANT_ID'
                    }
                 }
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
