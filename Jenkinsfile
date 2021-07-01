#!/usr/bin/env groovy
import org.apache.commons.lang.StringUtils
import java.util.regex.Matcher
import java.util.regex.Pattern

pipeline {
    agent {
		node {
			label "apiserver"
	    }
	}
	parameters {
		string(name: 'REPO_NAME', defaultValue: "https://github.com/ganeshcpote/jmeter.git", description: "Repository name to connect")
		string(name: 'BRANCH_NAME', defaultValue: "main", description: "Repository Branch name to push")
		booleanParam(name: 'RUN_LINT', defaultValue: true, description: 'Run Lint')
		booleanParam(name: 'RUN_UNITTESTS', defaultValue: true, description: 'Run Unittests')
		booleanParam(name: 'DEPLOY_CODE', defaultValue: false, description: 'Deploy Application')
		booleanParam(name: 'RUN_API', defaultValue: true, description: 'Run JMeter API Automation Suite')
		string(name: 'DEPLOY_SERVER', defaultValue: "10.166.168.8", description: 'Deployment Server')
	}
    stages {
        stage("SCM Checkout") {
            steps {				
				script {
					sh "git config --global http.sslVerify false"
				}
				git branch: params.BRANCH_NAME,
				credentialsId: 'github-token',
				url: params.REPO_NAME				
            }
        }		
		stage('Run API Testing') {
            steps {                
				script {
					sh '/root/apache-jmeter-5.4.1/bin/jmeter.sh  -n -t api.jmx -l lwm-api.csv -e -o ${BUILD_NUMBER}_htmlreport -Jserver ${DEPLOY_SERVER}'
				}                
            }
			post {
              always {
                script { 
					archive includes: '${BUILD_NUMBER}_htmlreport/*.*'
					publishHTML (target: [
					  allowMissing: false,
					  alwaysLinkToLastBuild: false,
					  keepAll: true,
					  reportDir: "${BUILD_NUMBER}_htmlreport",
					  reportFiles: 'index.html',
					  reportName: "API Execution Report"
					])                                   
                }
              }
            }
        }
    }
    post {        
        cleanup {
            echo 'One way or another, I have finished'
            deleteDir() /*clean up our workspace */
        }
    }
}

def findPattern(String regex, Boolean failIfFound, Boolean enabled) {
  if (enabled) {
    echo "find pattern Started"
    def logs = currentBuild.rawBuild.getLog(10000).join('\n')
    //echo "$logs"
    Pattern pattern = Pattern.compile(regex)
    Matcher matcher = pattern.matcher(logs)
    // Check all occurrences
    if (matcher.find()) {
      if (failIfFound) {
        echo "Match found, marking build as failed"
        currentBuild.result = 'FAILED'
      } else {
        echo "Match found, verification passed"
      }
    } else {
      if (!failIfFound) {
        echo "no match found, marking build as failed"
        currentBuild.result = 'FAILED'
      } else {
        echo "No Match found, verification passed"
      }
    }
  } else {
    echo "Pattern Match Disabled"
  }
}

def zip_artifacts(){
	zip zipFile: 'test.zip', archive: false, dir: 'archive'

}

def notifyBuild(String buildStatus = 'STARTED') {
    // build status of null means successful
  buildStatus = buildStatus ?: 'SUCCESSFUL'
  
  // Default values
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"
  def details = """<p>${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
  <p>Check console output at "<a href="${env.BUILD_URL}">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>"""

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESSFUL') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }

  //slackSend (color: colorCode, message: summary)

  emailext(
    subject: subject,
	mimeType: 'text/html',
	body: '''$DEFAULT_CONTENT''',
    to: "${DEVELOPMENT_TEAM_EMAIL}",
	from: "no-reply-lwm@ril.com",
	replyTo: 'ganesh.pote@ril.com',
	recipientProviders: [developers(), requestor()]
  )
}

def replace_variable(String oldText, String newText) {
	def text = readFile file: "start.sh"
	text = text.replaceAll("%${oldText}%", "${newText}")
	writeFile file: "start.sh", text: text
}
