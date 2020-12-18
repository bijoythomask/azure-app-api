node {

  stage('init') {
    checkout scm
  }

  stage('build') {
        sh '''
            mvn clean package
        '''

  }

  stage('Push to ACR') {
      withCredentials([azureServicePrincipal('azure_service_principal')]) {
        // login to Azure
        sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
          az acr login -n delsreg
          docker build --tag delsreg.azurecr.io/azure-app-api:latest .
          docker push delsreg.azurecr.io/azure-app-api:latest
        '''
      }
    }

  stage('deploy') {
    def resourceGroup = 'dels-jenkins-rg'
    def webAppName = 'dels-docker-app'
    def registryServer = 'delsreg.azurecr.io'
    def imageTag = sh script: 'git describe | tr -d "\n"', returnStdout: true
    def imageName = "$registryServer/calculator"

    withCredentials([azureServicePrincipal('azure_service_principal')]) {
        // login to Azure
        sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
          az webapp restart --name $dels-docker-app --resource-group $dels-jenkins-rg
        '''
    }

  }
}