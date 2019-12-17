pipeline {
    environment {
        registry = "garreeoke/person-api"
        registryCredential = "dockerhub"
        dockerImage = ""
    }
    agent any
    stages {
        stage ('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                ''' 
            }
        }

        stage ('Code Build') {
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true package'
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
