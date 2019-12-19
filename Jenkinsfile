#!/usr/bin/env groovy

def label = "docker-jenkins-${UUID.randomUUID().toString()}"
def home = "/home/jenkins/agent"
def workspace = "${home}/workspace/build-docker-jenkins"
def workdir = "${workspace}/src/localhost/docker-jenkins/"

def ecrRepoName = "garreeoke"
def tag = "person-api:" + "${BUILD_NUMBER}"
def repo = "${ecrRepoName}" + "/" + "$tag"
def dockerPwd = "Niners2019"

podTemplate(label: label,
        containers: [
                containerTemplate(name: 'jnlp', image: 'jenkins/jnlp-slave:alpine'),
                containerTemplate(name: 'maven', image: 'maven:3.6.3-ibmjava-8-alpine', command: 'cat', ttyEnabled: true),
                containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true, privileged: true),
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
  	
	    stage ('Code Build') {
              container('maven') {
                sh 'mvn -Dmaven.test.failure.ignore=true package'
              }
            }
            stage('Docker Build') {
                container('docker') {
                    echo "Building docker image... $repo"
                    sh "docker build -t $repo --build-arg JARFILE=person-0.0.1-SNAPSHOT.jar ."
                }
            }
            stage ('Docker Publish') {
              container('docker') {
                 sh "docker login -u garreeoke -p $dockerPwd" 
                 sh "docker push $repo"
                }
            }
            stage ('Remove Unused docker image') {
              container('docker') {
                sh "docker rmi $repo"
              }
            }
            stage ('Checkin Deployment Yaml') {
	      sh 'sed -i -E "s/person-api:.*/$tag/" deployment.yml'
              sh "git add deployment.yml"
              sh "git commit -m 'jenkins update'"
              timeout(time: 3, unit: "MINUTES") {
                checkin scm
              }
	    }
        }
    }
}
