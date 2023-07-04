import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles) {
    if (p['publishMethod'] == 'FTP') {
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
    }
  }
}

pipeline {
  agent any
  
  stages {
    stage('init') {
      steps {
        checkout scm
      }
    }
    
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
    
    stage('build') {
      steps {
        sh 'mvn clean package'
      }
    }
    
    stage('deploy') {
      steps {
        def resourceGroup = 'meghana_group-ba11'
        def webAppName = 'meghana'
        
        // Login to Azure
        withCredentials([usernamePassword(credentialsId: 'testing', passwordVariable: 'bb130157-e2d2-4c4a-8836-98479117b8c6', usernameVariable: '73f26c7b-ddd4-4181-9719-d86f8d1536e9')]) {
          sh '''
            az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
            az account set -s $AZURE_SUBSCRIPTION_ID
          '''
        }
        
        // Get publish settings
        def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
        def ftpProfile = getFtpPublishProfile(pubProfilesJson)
        
        // Upload package
        sh "curl -T target/calculator-1.0.war ${ftpProfile.url}/webapps/ROOT.war -u '${ftpProfile.username}:${ftpProfile.password}'"
        
        // Log out from Azure
        sh 'az logout'
      }
    }
  }
}
