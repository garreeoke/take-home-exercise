pipeline {
    environment {
        registry = "garreeoke/person-api"
        registryCredential = "dockerhub"
        dockerImage = ""
    }
    agent {
        docker {
            image 'maven:3-alpine' 
            args '-v /root/.m2:/root/.m2' 
        }
    }
    stages {
        stage ('Code Build') {
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true package'
            }
            post {
                success {
                    junit 'target/surefire-reports/**/*.xml' 
                }
            }
        }
        stage ('Docker Build') {
            steps {
                script {
                    dockerImage = docker.build (registry + ":${BUILD_NUMBER}", "--build-arg JARFILE=person-0.0.1-SNAPSHOT.jar .")
                }
            }
        }
        stage ('Docker Publish') {
            steps {
                script {
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push()
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
