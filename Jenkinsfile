/**
 * Copyright 2018, Boundless, https://boundlessgeo.com - All Rights Reserved
 * Unauthorized copying of this file, via any medium is strictly prohibited
 * Proprietary and confidential
 */

node {
  withCredentials([string(credentialsId: 'boundlessgeoadmin-token', variable: 'GITHUB_TOKEN'),
  string(credentialsId: 'sonar-jenkins-pipeline-token', variable: 'SONAR_TOKEN'),
  string(credentialsId: 'QUAY_SECRET', variable: 'QUAY_SECRET'),
  string(credentialsId: 'CF_API_URL' variable: 'CF_API_URL'),
  string(credentialsId: 'CF_APPS_USER' variable: 'CF_APPS_USER'),
  string(credentialsId: 'CF_APPS_PASSWORD' variable: 'CF_APPS_PASSWORD'),
  string(credentialsId: 'CF_APPS_ORG' variable: 'CF_APPS_ORG')]) {

    currentBuild.result = "SUCCESS"

    try {
      stage('Checkout'){
        checkout scm
          echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"
      }

      stage('Setup'){
        sh """
            docker login quay.io -u "boundlessgeo+jenkins" -p $QUAY_SECRET
        """
      }

      stage('Create Login') {
        sh """
          echo "cf login -a $CF_API_URL --skip-ssl-validation -u $CF_APPS_USER -p $CF_APPS_PASSWORD -o $CF_APPS_ORG" >> scripts/cf-login && cat scripts/cf-login
        """
      }

    }
    catch (err) {

      currentBuild.result = "FAILURE"
        throw err
    } finally {
      // Success or failure, always send notifications
      echo currentBuild.result
      notifyBuild(currentBuild.result)
    }

  }
}

// Slack Integration

def notifyBuild(String buildStatus = currentBuild.result) {

  // generate a custom url to use the blue ocean endpoint
  def jobName =  "${env.JOB_NAME}".split('/')
  def repo = jobName[0]
  def pipelineUrl = "${env.JENKINS_URL}blue/organizations/jenkins/${repo}/detail/${env.BRANCH_NAME}/${env.BUILD_NUMBER}/pipeline"
  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}\nJob: ${env.JOB_NAME}\nBuild: ${env.BUILD_NUMBER}\nJenkins: ${pipelineUrl}\n"
  def summary = (env.CHANGE_ID != null) ? "${subject}\nAuthor: ${env.CHANGE_AUTHOR}\n${env.CHANGE_URL}\n" : "${subject}"

  // Override default values based on build status
  if (buildStatus == 'SUCCESS') {
    colorName = 'GREEN'
    colorCode = '#228B22'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary, channel: '#bosh-release-bots')
}
