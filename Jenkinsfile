pipeline {
      agent any

      stages {
          stage('init') {
            steps {
                checkout scm
            }
          }
          stage('build') {
            steps{
                //sh 'mvn clean package'
                withCredentials([azureServicePrincipal('azure_service_principal')]) {
                  // login to Azure
                  sh '''
                    az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
                    az account set -s $AZURE_SUBSCRIPTION_ID
                  '''
                  sh 'az acr login -n delsreg && mvn compile jib:build'
                }
            }
          }
          stage('deploy') {
            environemnt {
                CLIENT_ID = sh (script:'az webapp identity assign --resource-group dels-jenkins-rg --name azure-app-api --query principalId --output tsv',, returnStdout: true).trim()
            }
            steps {
                withCredentials([azureServicePrincipal('azure_service_principal')]) {
                      // login to Azure
                      sh '''
                        az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
                        az account set -s $AZURE_SUBSCRIPTION_ID
                      '''
                      // Set default resource group name and service name. Replace <resource group name> and <service name> with the right values
                      sh 'az configure --defaults group=dels-jenkins-rg'
                      //sh 'az configure --defaults spring-cloud=<service name>'
                      // Deploy applications
                      //sh 'az spring-cloud app deploy -n gateway --jar-path ./gateway/target/gateway.jar'
                      //sh 'az spring-cloud app deploy -n account-service --jar-path ./account-service/target/account-service.jar'
                      //sh 'az spring-cloud app deploy -n auth-service --jar-path ./auth-service/target/auth-service.jar'

                      sh 'az appservice plan create --name AppSvc-Dels-plan --resource-group dels-jenkins-rg --is-linux'

                      sh 'az webapp create --resource-group dels-jenkins-rg --plan AppSvc-Dels-plan --name azure-app-api --deployment-container-image-name delsreg.azurecr.io/azure-app-api:latest'

                      sh 'az webapp config appsettings set --resource-group dels-jenkins-rg --name azure-app-api --settings SERVER_PORT=8000'



                      sh 'az role assignment create --assignee $CLIENT_ID --scope /subscriptions/$AZURE_SUBSCRIPTION_ID/resourceGroups/dels-jenkins-rg/providers/Microsoft.ContainerRegistry/registries/delsreg --role AcrPull'

                      sh 'az logout'

                    }
                }
          }
      }
}
