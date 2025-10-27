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
            
            # 确保使用正确的 Java 配置
            az webapp config set --name xinran-jenkins-webapp-2025 --resource-group jenkins-get-started-rg --java-version 11 --java-container Tomcat --java-container-version 9.0
            
            # 重启应用清除旧部署
            az webapp restart --name xinran-jenkins-webapp-2025 --resource-group jenkins-get-started-rg
            sleep 30
            
            # 检查构建生成的文件
            echo "=== 检查构建产物 ==="
            ls -la target/
            
            # 部署正确的 WAR 文件 (ROOT.war)
            az webapp deploy --resource-group jenkins-get-started-rg --name xinran-jenkins-webapp-2025 --src-path target/ROOT.war --type war --target-path ROOT.war
            
            echo "部署完成，等待应用启动..."
            sleep 90
        '''
    }
}

stage('verify') {
    sh '''
        echo "=== 测试应用 ==="
        # 等待应用完全启动
        sleep 60
        
        # 测试应用
        curl -f -v "http://xinran-jenkins-webapp-2025.azurewebsites.net/api/calculator/ping" || echo "第一次尝试失败，等待重试..."
        
        # 如果第一次失败，等待更长时间再试
        sleep 30
        curl -f -v "http://xinran-jenkins-webapp-2025.azurewebsites.net/api/calculator/ping" || echo "应用仍然没有响应"
        
        echo "=== 测试完成 ==="
    '''
}
  }
}
