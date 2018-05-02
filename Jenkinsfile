#!groovy​
// Include Xeno Global Library
@Library(['XENO_JENKINS', 'XENO_DEPLOY']) _
def getLabel() {
    // Which server to run this on.
    wrap([$class: 'ConsulKVReadWrapper',
      reads: [
      [debugMode: 'DISABLED', envKey: 'new_label', key: "drupal7/${getSitename()}/server"]
      ]]) {
      return env.new_label
    }
}
// Choose the site name based on git name and if it is a Pull Request or branch.
def getSitename() {
    SITENAME = "test-jenkins"
    // env.GIT_URL[env.GIT_URL.lastIndexOf('/')+1..-5]
    if (env.CHANGE_BRANCH && !env.CHANGE_FORK){
             return "${SITENAME}-${env.CHANGE_BRANCH.toLowerCase()}"
         }
         else{
             return "${SITENAME}-${env.BRANCH_NAME.toLowerCase()}"
         }
}
def getSlackname() {
    return sh (
          script: 'git --no-pager show -s --format=%ae',
          returnStdout: true
    ).trim()
}
pipeline {
  environment {
    // Slack IDs for notification.
    X_SLACK_NOTIFY = "<@U0CNWRWGY|albert> <@U043NEGJC|michaelpporter>"
    // Slack Channel
    X_SLACK_CHANNEL = "#jenkins-ci"
  }
  agent {
    node {
      label "${getLabel()}"
      customWorkspace "/var/www/${getSitename()}"
    }
  }
  options {
    // do not run more than one build per branch
      disableConcurrentBuilds()
    // lock based on branch name, pull requests use branch name
      // lock resource: "${getSitename()}"
      lock(resource: "test-jenkins", inversePrecedence: true)
    // timeout after 8 hours
      timeout(time: 8, unit: 'HOURS')
    // keep 7 jobs
      buildDiscarder(logRotator(numToKeepStr: '7'))
      skipDefaultCheckout()
  }
  stages {
    stage("Checkout") {
      steps {
        checkout scm
      }
    }
    stage("Setup") {
      when {
        // Only build if the site is new
        expression {
          return !fileExists("/var/www/${getSitename()}/web/.htaccess")
        }
      }
      steps {
        echo "Build variables"
        echo "${getSitename()}"
        echo env.X_DB_BACKUP
        echo env.X_DB_USER
        echo env.X_LIVE_DOMAIN
        echo "${getLabel()}"
        echo "Build variables"
      }
    }
    stage("Drupal") {
      steps {
        sh """
            printenv
            sleep 10
          """
      }
    }
    stage ('Deploy Code') {
        when {
            branch 'master'
        }
      steps {
        sh """
            printenv
            sleep 120
          """
      }
    }
  }
  post {
    success {
       slackNotify("Build Compete. https://${getSitename()}.xenostaging.com", "${X_SLACK_CHANNEL}", 'good')
    }
    failure {
       slackNotify("Build Failed. https://${getSitename()}.xenostaging.com", "${X_SLACK_CHANNEL}", 'danger', "${X_SLACK_NOTIFY}", "${getSlackname()}")
    }
    unstable {
       slackNotify("Build is unstable. https://${getSitename()}.xenostaging.com", "${X_SLACK_CHANNEL}", 'warning', "${X_SLACK_NOTIFY}", "${getSlackname()}")
    }
  }
}
