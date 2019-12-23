pipeline {
    environment {
        registry = "garreeoke/person-api"
        registryCredential = "dockerhub"
        label = "docker-jenkins-${UUID.randomUUID().toString()}"
        home = "/home/jenkins/agent"
        workspace = "${home}/workspace/armory1"
        workdir = "${workspace}/src/localhost/docker-jenkins/"
        tag = "person-api:" + "${BUILD_NUMBER}"
        repo = "garreeoke/" + "$tag"
        dockerPwd = "Niners2019"
        GIT_AUTH = credentials('gareeoke-github')
    }
    agent {
     kubernetes {
        yamlFile 'kubepod.yml'
      } 
    }
      stages {
        stage('Checkout') {
          steps {
               timeout(time: 3, unit: 'MINUTES') {
               checkout scm
               }
          }
        }
        stage ('Code Build') {
            steps {
              container('maven') {
                  sh 'mvn -Dmaven.test.failure.ignore=true package'
                }
            }
        }
        stage ('Docker Build') {
            steps {
              container('docker') {
                echo "Building docker image ... $repo"
                sh "docker build -t $repo --build-arg JARFILE=person-0.0.1-SNAPSHOT.jar ."
              }
            }
        }
        stage ('Docker Publish') {
            steps {
              container('docker') {
                sh "docker login -u garreeoke -p $dockerPwd" 
               sh "docker push $repo"
              }
            }
        }
        stage ('Checkin') {
            steps {
                  sh('''
                     echo "USGR: $GIT_AUTH_USR"
                     git checkout --track origin/armory
                     sed -i -E "s/person-api:.*/$tag/" deployment.yml
                     git config --global user.email "garreesett@gmail.com"
                     git config --global user.name "garreeoke"
                     git add deployment.yml 
                     git commit -m "[Jenkins CI] Add build file"
                     git config --local credential.helper "!f() { echo username=\\$GIT_AUTH_USR; echo password=\\$GIT_AUTH_PSW; }; f"
                     git push 
                 ''')
            }
        }
      }
}
