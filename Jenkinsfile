
def SendSlackNotification(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESS'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESS') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }
// Send notifications
  slackSend (color: colorCode, message: summary)
}


node{
  
echo "Build Number is : ${env.BUILD_NUMBER}"
echo "Job Name is : ${env.JOB_NAME}"
def mavenHome = tool name: 'maven3.8.5'

try{  
  //Get the code from Github
stage('CheckoutCode'){
git branch: 'development', credentialsId: '9750b4d2-c9fa-4d69-9a28-dfdda803a4ce', url: 'https://github.com/bmaheswarn/maven-web-application.git'
}
//Do the build using Maven
stage('Build'){
sh "${mavenHome}/bin/mvn clean package"
}

//Execcute SonarQube report
stage('ExecuteSonarQubeReport'){
sh "${mavenHome}/bin/mvn sonar:sonar"
}

//Uploan into Artifact Repository
stage('UploadInArtifactRepository'){
sh "${mavenHome}/bin/mvn deploy"
}

//Application Deploy into Tomcat Server

stage('DeployApplicationIntoTomcarServer'){
sshagent(['ec2-user']) {
sh "scp -o StrictHostKeyChecking=no target/maven-web-application.war ec2-user@3.145.37.131:/opt/apache-tomcat-9.0.62/webapps"
}
}

}//try closing
  catch{
    currentBuild.result = "FAILED"
  }
  finally{
  SentSlackNotifications(currentBuild.result)
  }
}//node closing
