pipeline {
    agent any

    environment {
        AZURE_SERVICE_PRINCIPAL_ID = credentials('fc5e47eb-ddcb-4481-beab-1acefeb33393')
        AZURE_SERVICE_PRINCIPAL_SECRET = credentials('5f761a78-8757-4beb-9cfc-ec29f1caef52')
        AZURE_TENANT_ID = credentials('c81613f1-8384-4d3d-9416-1a0259a03d57')
    }
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/kimin-park/jenkins-cicd.git']]])
            }
        }
        stage('Build and Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')])
                script {
                    def dockerImage = 'lkasd7512/nginx-proxy'
                    def dockerTag = '1.0'
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        def builtImage = docker.build("$dockerImage:$dockerTag", '.')
                        builtImage.push()
                    }
                }
            }
        }
        stage('Authenticate to Azure') {
            steps {
                script {
                    withCredentials([azureServicePrincipal('azure-jenkins')]) {
                        def azureCredentials = azureServicePrincipal('azure-jenkins')
                        def az = azureCLI(credentialsId: azureCredentials.id)
                        az.withAuth {
                            sh 'az login --service-principal -u $AZURE_SERVICE_PRINCIPAL_ID -p $AZURE_SERVICE_PRINCIPAL_SECRET --tenant $AZURE_TENANT_ID'
                        }
                    }
                }
            }
        }
        stage('Deploy to AKS') {
            steps {
                withAzureCliInstallation('default') {
                    sh '''
                        az aks get-credentials --resource-group project2-msa-rg --name msacluster
                        kubectl apply -f nginx-proxy.yaml
                    '''
                }
            }
        }
    }
}
