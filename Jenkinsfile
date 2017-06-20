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
  sh "docker container prune -f"
  sh 'docker images --filter "dangling=true" | { grep -e "\\(weeks\\|mongths\\)" || true; } > dangling-docker-images.txt'
  sh 'docker images | { grep -e "\\(pena\\|distillery\\|smartlyv1\\)" || true; } | { grep -e "\\(weeks\\|mongths\\)" || true; } > smartly-images.txt'
  sh script: 'docker rmi "$(cat dangling-docker-images.txt smartly-images.txt | uniq | awk \'{ print $3 }\')"', returnStatus: true
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
parallel jobs
