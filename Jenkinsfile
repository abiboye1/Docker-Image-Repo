pipeline {
    agent any

    environment {
        VERSION = "1.0.${BUILD_NUMBER}"
        PATH = "${PATH}:${getSonarPath()}:${getDockerPath()}"
    }

    stages {
        stage ('Sonarcube Scan') {
        steps {
         script {
          scannerHome = tool 'sonarqube'
        }
        withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]){
        withSonarQubeEnv('SonarQubeScanner') {
          sh " ${scannerHome}/bin/sonar-scanner \
          -Dsonar.projectKey=CliXX-App-Abib   \
          -Dsonar.login=${SONAR_TOKEN} "
        }
        }
        }

}

 stage('Quality Gate') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
            }
            }
        }


  stage ('Build Docker Image and Push To ECR') {
          steps {
              sh "/usr/bin/docker build . -t clixx-image:$VERSION "
          }
        }

  stage ('Starting Docker Image for Testing') {
          steps {
              sh "/usr/bin/docker run --name clixx-cont-$VERSION  -p 80:80 -d clixx-image:$VERSION"
          }
        }


  stage ('Deployment Tear Down Prompt ') {
              steps {
                script {
                def userInput = input(id: 'confirm', message: 'Please Test CliXX Image Deployment. Should I delete now?', parameters: [ [$class: 'BooleanParameterDefinition', defaultValue: false, description: 'Tear Down App', name: 'confirm'] ])
             }
             sh " docker stop clixx-cont-$VERSION "
             sh " docker rm  clixx-cont-$VERSION "
           }
        }

  stage ('Log Into ECR and push the newly created Docker') {
          steps {
             script {
                def userInput = input(id: 'confirm', message: 'Push Image To ECR?', parameters: [ [$class: 'BooleanParameterDefinition', defaultValue: false, description: 'Push to ECR?', name: 'confirm'] ])
             }
              sh '''
                aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 767397944080.dkr.ecr.us-east-1.amazonaws.com/clixx-repository
                docker tag clixx-image:$VERSION 767397944080.dkr.ecr.us-east-1.amazonaws.com/clixx-repository:clixx-image-$VERSION
                docker tag clixx-image:latest 767397944080.dkr.ecr.us-east-1.amazonaws.com/clixx-repository:clixx-image-$VERSION
                docker push 767397944080.dkr.ecr.us-east-1.amazonaws.com/clixx-repository:clixx-image-$VERSION
              '''
          }
        }

    }

}



def getSonarPath(){
        def SonarHome= tool name: 'sonarqube', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
        return SonarHome
    }
def getDockerPath(){
        def DockerHome= tool name: 'docker-inst', type: 'dockerTool'
        return DockerHome
    }
    

