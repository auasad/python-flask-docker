pipeline {
  environment {
    registry = "auasad/python-cicd"
    registryCredential = 'dockerhub'
    dockerImage = ''
  }
  agent any
  stages {
    stage('Cloning Git') {
      steps {
        git 'https://github.com/auasad/python-flask-docker.git'
      }
    }
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
        }
      }
    }
    stage('Deploy Image') {
      steps{
        script {
          docker.withRegistry( '', registryCredential ) {
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
    stage('Stopping & remove Old Image'){
      steps{
        sshagent(['DockerDevServer']) {
          sh 'ssh -o StrictHostKeyChecking=no asad@10.128.0.3 docker stop my-python-app || true'
          sh 'ssh -o StrictHostKeyChecking=no asad@10.128.0.3 docker rm my-python-app || true'
        }  
      }
   }
    stage('Deploy Container on Dev') {
      steps{
      script {
        docker.withRegistry( '', registryCredential ) {
          def dockerRun = 'docker run -p 8080:8080 -d --name my-python-app auasad/python-cicd:${BUILD_NUMBER}'
            sshagent(['instance-uat']) {
              sh "ssh -o StrictHostKeyChecking=no asad@10.128.0.3 ${dockerRun}"
           } }
        }
      }
    }
}
}
