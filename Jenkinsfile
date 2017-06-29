#!/usr/bin/env groovy

// The ArrayList of slaves is not serializable, so fetching them should be marked as @NonCPS so that
// no attempt is made to serialize and save the local state of the function. See here for details:
// https://github.com/jenkinsci/pipeline-plugin/blob/master/TUTORIAL.md#serializing-local-variables
@NonCPS
def getSlaves() {
  def slaves = []
  for (hudson.model.Slave it : jenkins.model.Jenkins.instance.slaves) {
    slaves << it.name
  }
  return slaves
}

def runDockerCleanup() {
  sh "docker system prune -f || true"
  sh 'docker images | { grep -e "\\(pena\\|distillery\\|smartlyv1\\)" || true; } | { grep -e "\\(weeks\\|months\\)" || true; } > smartly-images.txt'
  sh "cat smartly-images.txt | awk '{ print \$3 }' | uniq | xargs docker rmi -f || true"
}

properties([
  [$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator', daysToKeepStr: '7']],
  pipelineTriggers([cron('H 4 * * 1')]),
])

// Run a command on each slave in parallel
def jobs = [:]
for (String slave : getSlaves()) {

  // Create a closure for each slave and put it in the map of jobs
  jobs[slave] = {
    node(slave) {
      runDockerCleanup()
    }
  }
}

jobs['master'] = {
  node('master') {
    runDockerCleanup()
  }
}

// Run the closures in parallel
stage('Docker Cleanup') {
  parallel jobs
}
