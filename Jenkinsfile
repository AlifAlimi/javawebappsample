import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  stage('init') {
    checkout scm
  }
  
  stage('build') {
    sh 'mvn clean package'
  }
  
  stage('deploy') {
    def resourceGroup = 'QuickstartJenkins-rg' 
    def webAppName = 'jenkins-app'
    // login Azure
    withCredentials([azureServicePrincipal('1a57b7e8-b624-4564-a519-44bd5ea25cee')]) {
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
