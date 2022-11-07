@Library('slack') _

pipeline {
  agent any
 environment { 
        registry = "testvikrams1/numeric-app" 
        registryCredential = 'dockerhub_id' 
        dockerImage = '' 
        deploymentName = "devsecops"
        containerName = "devsecops-container"
        serviceName = "devsecops-svc"
        imageName = "testvikrams1/numeric-app:${GIT_COMMIT}"
        applicationURL = "http://devsecops-demov.eastus.cloudapp.azure.com"
        applicationURI = "/increment/99"
    }


 
  
   stages {
    stage('Testing Slack') {
      steps {
        sh 'exit 1'
      }
    }

  }

  post {
    always {
      junit 'target/surefire-reports/*.xml'
      jacoco execPattern: 'target/jacoco.exec'
      pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
      dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
      publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'owasp-zap-report', reportFiles: 'zap_report.html', reportName: 'OWASP', reportTitles: 'OWASP'])
           // Use sendNotifications.groovy from shared library and provide current build result as parameter    
    //  sendNotification currentBuild.result
    }

    success {
      script {
        /* Use slackNotifier.groovy from shared library and provide current build result as parameter */
        env.failedStage = "none"
        env.emoji = ":white_check_mark: :tada: :thumbsup_all:"
        sendNotification currentBuild.result
      }
    }

    // failure {

    // }
  
}
}