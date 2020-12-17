node {
  stage('init') {
    checkout scm
  }

  stage('build') {
    sh 'mvn clean package'
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
        dockerImageName: imageName,
        dockerImageTag: imageTag,
        dockerRegistryEndpoint: [credentialsId: 'delsreg', url: "http://$registryServer"]

  }
}