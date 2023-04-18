 node {
 withEnv(['AZURE_SUBSCRIPTION_ID=',
         'AZURE_TENANT_ID1=']) {
     stage('Init') {
         cleanWs()
         checkout scm
     }

     stage('Build') {
           withCredentials([usernamePassword(credentialsId: 'azuresp', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
           pwsh '''
             Write-Host "started"
             az login --service-principal -u %AZURE_CLIENT_ID% -p %AZURE_CLIENT_SECRET% -t %AZURE_TENANT_ID1%
             az account set -s %AZURE_SUBSCRIPTION_ID%
             $User = $env:AZURE_CLIENT_ID
             $tenant = $env:AZURE_TENANT_ID1
             $Secret = $env:AZURE_CLIENT_SECRET
             Write-Host "started is $User"
             Write-Host "started is $tenant"
             $PWord = ConvertTo-SecureString -String $Secret -AsPlainText -Force
             $Credential = New-Object -TypeName "System.Management.Automation.PSCredential" -ArgumentList $User, $PWord
             Connect-AzAccount -Credential $credential -ServicePrincipal -TenantId $tenant
         '''
         }
         pwsh '''
         
                        .\\\\Scripts\\\\Deploy\\\\Build-DeploymentPlans.ps1 `
                        -pacEnvironmentSelector "epac-dev" `
                        -definitionsRootFolder "./StarterKit/Definitions" `
                        -devOpsType "ado" `
                        -InformationAction Continue'''
     }

     stage('Publish') {
         def RESOURCE_GROUP = '<resource_group>' 
         def FUNC_NAME = '<function_app>'
         // login Azure
         withCredentials([usernamePassword(credentialsId: 'azuresp', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
         sh '''
             az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
             az account set -s $AZURE_SUBSCRIPTION_ID
         '''
         }
         sh 'cd $PWD/target/azure-functions/odd-or-even-function-sample && zip -r ../../../archive.zip ./* && cd -'
         sh "az functionapp deployment source config-zip -g $RESOURCE_GROUP -n $FUNC_NAME --src archive.zip"
         sh 'az logout'
         }
     }
 }
