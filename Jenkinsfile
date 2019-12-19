#!/usr/bin/env groovy

def label = "docker-jenkins-${UUID.randomUUID().toString()}"
def home = "/home/jenkins"
def workspace = "${home}/workspace/build-docker-jenkins"
def workdir = "${workspace}/src/localhost/docker-jenkins/"

def ecrRepoName = "garreeoke"
def tag = "${ecrRepoName}" + "/person-api:" + "${BUILD_NUMBER}"

podTemplate(label: label,
        containers: [
                containerTemplate(name: 'jnlp', image: 'jenkins/jnlp-slave:alpine'),
                containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true, privileged: false),
            ],
            volumes: [
                hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'),
            ],
        ) {
    node(label) {
        dir(workdir) {
            stage('Checkout') {
                timeout(time: 3, unit: 'MINUTES') {
                    checkout scm
                }
            }

            stage('Docker Build') {
                container('docker') {
                    echo "Building docker image... $tag"
                    sh "docker build -t $tag -f jenkins-docker/Dockerfile ."
                }
            }
        }
    }
}
