node {
  stage('Preparation') { // for display purposes
    // Get some code from a GitHub repository
    git 'https://github.com/confluentinc/common.git'
  }
  stage('Build') {
    // Run the maven build
    sh "mvn --batch-mode -Pjenkins clean install dependency:analyze site"
  }
  stage('Deploy') {
    withMaven(
      globalMavenSettingsConfig: 'jenkins-maven-global-settings',
    ) {
      // Run the maven build
      sh "mvn --batch-mode -Pjenkins $params.deploy_options deploy -DskipTests"
    } // withMaven will discover the generated Maven artifacts, JUnit Surefire & FailSafe reports and FindBugs reports
  }
  stage('Notify') {
    junit '**/target/surefire-reports/TEST-*.xml'
    step([$class: 'hudson.plugins.checkstyle.CheckStylePublisher', pattern: '**/target/checkstyle-result.xml', unstableTotalAll:'0'])
    step([$class: 'hudson.plugins.findbugs.FindBugsPublisher', pattern: '**/findbugsXml.xml'])
    archive 'target/*.jar'

    switch(currentBuild.currentResult) {
      case 'SUCCESS':
        if (currentBuild.previousBuild != null && currentBuild.previousBuild.currentResult != 'SUCCESS') {
          slackSend(channel: '#what-a-time', color: 'good', message: "${env.JOB_NAME} - #[${env.BUILD_NUMBER}] Success <${env.BUILD_URL}|(Open)>", teamDomain: 'confluent')
        }
        break;
      case 'UNSTABLE':
        slackSend(channel: '#what-a-time', color: 'YELLOW', message: "${env.JOB_NAME} - #[${env.BUILD_NUMBER}] Unstable <${env.BUILD_URL}|(Open)>", teamDomain: 'confluent')
        break;
      case 'FAILURE':
        slackSend(channel: '#what-a-time', color: 'bad', message: "${env.JOB_NAME} - #[${env.BUILD_NUMBER}] Failure <${env.BUILD_URL}|(Open)>", teamDomain: 'confluent')
        break;
    }
  }
}