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
        '''
    }
    
    // 获取 FTP 发布配置文件
    def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName --xml", returnStdout: true
    
    // 解析 XML 获取 FTP 信息（需要安装 xml2 或使用其他方法解析）
    def ftpUrl = sh script: "echo '$pubProfilesJson' | grep -oP 'publishUrl=\"\\K[^\"]+' | head -1", returnStdout: true
    def ftpUser = sh script: "echo '$pubProfilesJson' | grep -oP 'userName=\"\\K[^\"]+' | head -1", returnStdout: true
    def ftpPass = sh script: "echo '$pubProfilesJson' | grep -oP 'userPWD=\"\\K[^\"]+' | head -1", returnStdout: true
    
    // 使用 FTP 上传
    sh """
        curl -T target/calculator-1.0.war "ftp://$ftpUrl/site/wwwroot/webapps/ROOT.war" --user "$ftpUser:$ftpPass"
    """
}
  }
}
