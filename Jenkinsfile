properties([pipelineTriggers([githubPush()])])

node{
    def mavenHome = tool name:'Maven-3.9.11' ,type: 'maven'
    def tomcatIp='13.60.65.217'
    def tomcatUser='ec2-user'
    try{
        stage('Checkout') {
            checkout([$class: 'GitSCM',  
                branches: [[name: '*/main']],
                extensions: [
                    [$class: 'WipeWorkspace'],
                    [$class: 'CloneOption', noTags: false, shallow: false, depth: 0, timeout: 10]
                ], 
                userRemoteConfigs: [[url: 'https://github.com/sachinthitame123/student-reg-webapp.git']]
            ])
        }

        stage("Maven build")
        {
            sh "${mavenHome}/bin/mvn clean package"
        }
        stage("Sonar")
        {
            withCredentials([string(credentialsId: 'sonarToken', variable: 'sonarToken')]) {
                sh "${mavenHome}/bin/mvn verify sonar:sonar -Dsonar.token=${sonarToken}"
            }
        }
         stage("Upload war file to Nexus")
        {
             sh "${mavenHome}/bin/mvn package deploy"
        }
        stage("Upload war file to Tomcat")
        {
            sshagent(['TomcatCredentials']) {
              sh "ssh -o StrictHostKeyChecking=no ${tomcatUser}@${tomcatIp} sudo systemctl stop tomcat"
              sh "sleep 20"
              sh "ssh -o StrictHostKeyChecking=no ${tomcatUser}@${tomcatIp} rm /opt/tomcat/webapps/student-reg-webapp.war"
              sh "scp -o StrictHostKeyChecking=no target/student-reg-webapp.war ${tomcatUser}@${tomcatIp}:/opt/tomcat/webapps/student-reg-webapp.war"
              sh "ssh -o StrictHostKeyChecking=no ${tomcatUser}@${tomcatIp} sudo systemctl start tomcat"
            }
        }
    }catch(err)
    {
        currentBuild.result='FAILURE'
        throw err
    }
    finally {
    def buildStatus = currentBuild.result ?: 'SUCCESS'

    // Define color, emoji, message
    def color = (buildStatus == 'SUCCESS') ? 'green' : 'red'
    def emoji = (buildStatus == 'SUCCESS') ? '✅' : '❌'
    def message = (buildStatus == 'SUCCESS') ?
        "${emoji} Build *SUCCESSFUL* for job: ${env.JOB_NAME} #${env.BUILD_NUMBER}" :
        "${emoji} Build *FAILED* for job: ${env.JOB_NAME} #${env.BUILD_NUMBER}"

    def emailBody = """
        <html>
            <body style="font-family:Arial, sans-serif; color:#333;">
                <h2 style="color:${color};">${emoji} ${buildStatus}</h2>
                <p><b>Project:</b> ${env.JOB_NAME}</p>
                <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
                <p><b>Build URL:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                <hr>
                <p>${message}</p>
            </body>
        </html>
    """

    emailext(
        subject: "Jenkins Build: ${buildStatus} - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: emailBody,
        mimeType: 'text/html',
        to: 'sachinthitame7350@gmail.com'
    )
    }
}   