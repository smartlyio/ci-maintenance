#!/usr/bin/env groovy

// The ArrayList of slaves is not serializable, so fetching them should be marked as @NonCPS so that
// no attempt is made to serialize and save the local state of the function. See here for details:
// https://github.com/jenkinsci/pipeline-plugin/blob/master/TUTORIAL.md#serializing-local-variables
@NonCPS
def getSlaves() {
    def slaves = []
    hudson.model.Hudson.instance.slaves.each {
        slaves << it.name
    }
    return slaves
}

properties([[$class: 'BuildDiscarderProperty',
                strategy: [$class: 'LogRotator', daysToKeepStr: '7']],
                pipelineTriggers([cron('H 4 * * 1')]),
                ])

// Run a command on each slave in parallel
def jobs = [:]
getSlaves().each {

    // Use a local variable to avoid closing over a reference to 'it',
    // the value of which changes on each iteration
    def slave = it

    // Create a closure for each slave and put it in the map of jobs
    jobs[slave] = {
        node(slave) {
            sh "docker container prune -f"
            sh 'docker images --filter "dangling=true" | grep -e "\(weeks\|mongths\)" > dangling-docker-images.txt'
            sh 'docker images | grep -e "\(pena\|distillery\|smartlyv1\)" | grep -e "\(weeks\|mongths\)" > smartly-images.txt'
            sh script: 'docker rmi "$(cat dangling-docker-images.txt smartly-images.txt | uniq | awk \'{ print $3 }\')"', returnStatus: true
            sh 'rm dangling-docker-images.txt smartly-images.txt'
        }
    }
}

// Run the closures in parallel
parallel jobs
