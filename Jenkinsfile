properties([parameters([string(name: 'NAME', defaultValue: 'test', description: 'Parameter for docker Name', trim: true)])])
pipeline {
  agent any
    options {
      buildDiscarder(logRotator(daysToKeepStr: '3', numToKeepStr: '10'))
      disableConcurrentBuilds()
    }
    tools {
      maven 'maven3'
    }
    environment {
      DOCKER_TAG = getVersion()

    }
    stages{
        stage('Maven Build'){
            steps{
                sh "mvn clean package"
            }
        }
        
        stage('Docker Build'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'dockerpwd', usernameVariable: 'dockeruser')]) {
                    sh '''
                    docker build . --tag ${dockeruser}/${NAME}:${DOCKER_TAG}
                    '''
                }
            }
        }
        
        stage('DockerHub Push'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'dockerpwd', usernameVariable: 'dockeruser')]) {
                    sh '''
                        docker login -u ${dockeruser} -p ${dockerpwd}
                        docker push ${dockeruser}/${NAME}:${DOCKER_TAG}
                    '''
                }
            }
        }
        
        stage('Docker Deploy'){
            steps{
                sh '''
                    docker image prune -f
                    docker stop dockerproject-api-1
                    docker stop dockerproject-worker-1
                    docker-compose up -d
                    echo 'docker compose deploy Completed' 
                '''
            }
        }
    }
}

def getVersion(){
    def commitHash = sh label: '', returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}
