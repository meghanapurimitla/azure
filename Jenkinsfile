import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

pipeline {
  agent any
  
  environment {
    AZURE_SUBSCRIPTION_ID = '80c89423-f05a-47c3-aa56-2bee83409fa0'
    AZURE_TENANT_ID = '1cee9c63-a1eb-4a3a-95e6-c4e16e9a128f'
  }
  
  stages {
    stage('init') {
      steps {
        checkout scm
      }
    }
    
    stage('build') {
      steps {
        sh 'mvn clean package'
      }
    }
    
    stage('deploy') {
      steps {
        script {
          def resourceGroup = 'meghana_group-ba11'
          def webAppName = 'meghana'
          
          // login Azure
withCredentials([azureServicePrincipal(credentialsId: 'testing',
                                    subscriptionIdVariable: '80c89423-f05a-47c3-aa56-2bee83409fa0',
                                    clientIdVariable: '73f26c7b-ddd4-4181-9719-d86f8d1536e9',
                                    clientSecretVariable: 'bb130157-e2d2-4c4a-8836-98479117b8c6',
                                    tenantIdVariable: '1cee9c63-a1eb-4a3a-95e6-c4e16e9a128f')]) {
    sh 'az login --service-principal -u $CLIENT_ID -p $CLIENT_SECRET -t $TENANT_ID'
}       }
          
          // get publish settings
          def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
          def ftpProfile = getFtpPublishProfile(pubProfilesJson)
          
          // upload package
          sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '${ftpProfile.username}:${ftpProfile.password}'"
          
          // log out
          sh 'az logout'
        }
      }
    }
  }
}
