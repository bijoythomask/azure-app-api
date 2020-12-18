node {
  stage('init') {
    checkout scm
  }

  stage('build') {
    withCredentials([azureServicePrincipal('azure_service_principal')]) {
      // login to Azure
      //         az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
      //         az account set -s $AZURE_SUBSCRIPTION_ID
      //         az acr login -n delsreg
      sh '''

        mvn compile jib:build
      '''
    }
  }

  stage('deploy') {
    def resourceGroup = 'dels-jenkins-rg'
    def webAppName = 'dels-docker-app'
    def registryServer = 'delsreg.azurecr.io'
    def imageTag = sh script: 'git describe | tr -d "\n"', returnStdout: true
    def imageName = "$registryServer/calculator"
    azureWebAppPublish azureCredentialsId: 'azure_service_principal',
        publishType: 'docker',
        resourceGroup: resourceGroup,
        appName: webAppName,
//         dockerImageName: imageName,
//         dockerImageTag: imageTag,
        dockerRegistryEndpoint: [credentialsId: 'delsreg']
  }
}