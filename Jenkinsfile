pipeline {
    environment {
        registry = "garreeoke/person-api"
        registryCredential = "dockerhub"
        dockerImage = ""
    }
    agent {
        kubernetes {
          label 'mypod'
          yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: maven:3.6.3-ibmjava-8-alpine
    command: ['cat']
    tty: true
    priviledged: true
    volumeMounts:
    - name: dockersock
      mountPath: /var/run/docker.sock
    - name: m2
      mountPath: /root/.m2
  volumes:
  - name: dockersock
    hostPath:
      path:/var/run/docker.sock
  - name: m2
    hostPath:
      path: /root/.m2
"""
        }
    }
    stages {
        stage ('Code Build') {
            steps {
                container('docker') {
                  sh 'mvn -Dmaven.test.failure.ignore=true package'
                }
            }
            post {
                success {
                    junit 'target/surefire-reports/**/*.xml' 
                }
            }
        }
        stage ('Docker Build') {
            steps {
                container('docker') {
                   script {
                       dockerImage = docker.build (registry + ":${BUILD_NUMBER}", "--build-arg JARFILE=person-0.0.1-SNAPSHOT.jar .")
                   }
                }
            }
        }
        stage ('Docker Publish') {
            steps {
                container('docker') {
                   script {
                       docker.withRegistry('', registryCredential) {
                        dockerImage.push()
                       }
                   }
                }
            }
        }
        stage('Remove Unused docker image') {
          steps{
            sh "docker rmi $registry:$BUILD_NUMBER"
          }
        }
    }
}
