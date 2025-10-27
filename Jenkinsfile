import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=8a72ad35-4b45-4b2a-bf3f-e4f7cfe21521',
        'AZURE_TENANT_ID=43b0ff79-3f17-4ad2-9fb1-593cd0fa446b']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
    def resourceGroup = 'jenkins-get-started-rg'
    def webAppName = 'xinran-jenkins-webapp-2025'
    
    withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
        sh '''
            az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
            az account set -s $AZURE_SUBSCRIPTION_ID
            
            # 现在部署 ROOT.war（Maven 生成的文件名）
            az webapp deploy --resource-group jenkins-get-started-rg --name xinran-jenkins-webapp-2025 --src-path target/ROOT.war --type war
            
            echo "部署完成，等待应用启动..."
            sleep 60
        '''
    }
}
  }
}
