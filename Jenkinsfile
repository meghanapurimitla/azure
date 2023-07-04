import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}


node {
  withEnv(['AZURE_SUBSCRIPTION_ID=80c89423-f05a-47c3-aa56-2bee83409fa0',
        'AZURE_TENANT_ID=1cee9c63-a1eb-4a3a-95e6-c4e16e9a128f']) {
    stage('init') {
      checkout scm
    }
    stages {
    stage('Azure Login') {
      steps {
        withAzureServicePrincipal(credentialsId: 'testing') {
          // Azure CLI commands or Azure CLI Jenkins plugin steps can be used here
          sh 'az login'
        }
      }
    }
    
    stage('Deploy to Azure App Service') {
      steps {
        withAzureAppService(credentialsId: 'testing', resourceGroup: 'meghana_group-ba11', appName: 'meghana') {
          // Azure CLI commands or Azure CLI Jenkins plugin steps to deploy to App Service
          sh 'az webapp deploy --name meghana --resource-group meghana_group-ba11 --src-path your-app-package.zip'
        }
      }
    }
  }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'meghana_group-ba11'
      def webAppName = 'meghana'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'testing', passwordVariable: 'bb130157-e2d2-4c4a-8836-98479117b8c6', usernameVariable: '73f26c7b-ddd4-4181-9719-d86f8d1536e9')]) {
       sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }
      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson
      // upload package
      sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      // log out
      sh 'az logout'
    }
  }
}
}
