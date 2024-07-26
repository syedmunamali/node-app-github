pipeline {
  agent any

  stages {
    stage('Build') {
      steps {
        script {
          docker.build("node-app:${env.BUILD_NUMBER}")
        }
      }
    }

    stage('Save Image') {
      steps {
        script {
          sh "docker save -o node-app_${env.BUILD_NUMBER}.tar node-app:${env.BUILD_NUMBER}"
        }
      }
    }

    stage('Transfer Image') {
      steps {
        sshPublisher(publishers: [
          sshPublisherDesc(
            configName: 'ubuntuserver',
            transfers: [
              sshTransfer(
                sourceFiles: "node-app_${env.BUILD_NUMBER}.tar",
                removePrefix: '',
                remoteDirectory: '', // No remote directory, using the home directory
                execCommand: '''
                  docker load -i node-app_${BUILD_NUMBER}.tar || { echo "Docker load failed"; exit 1; }
                  docker stop node-app-${BUILD_NUMBER} || true
                  docker rm node-app-${BUILD_NUMBER} || true
                  docker run -d -p 3000:3000 --name node-app-${BUILD_NUMBER} node-app:${BUILD_NUMBER} || { echo "Docker run failed"; exit 1; }
                '''
              )
            ]
          )
        ])
      }
    }
  }
}
