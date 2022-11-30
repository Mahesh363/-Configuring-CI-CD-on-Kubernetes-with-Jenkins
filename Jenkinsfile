pipeline {
  agent any
  stages {
    stage('Docker Build') {
      steps {
        bat "docker build -t kmlaydin/podinfo:${env.BUILD_NUMBER} ."
      }
    }
    stage('Docker Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
          bat "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
          bat "docker push kmlaydin/podinfo:${env.BUILD_NUMBER}"
        }
      }
    }
    stage('Docker Remove Image') {
      steps {
        bat "docker rmi kmlaydin/podinfo:${env.BUILD_NUMBER}"
      }
    }
    stage('Apply Kubernetes Files') {
      steps {
          withKubeConfig([credentialsId: 'kubeconfig']) {
          bat 'cat deployment.yaml | sed "s/{{BUILD_NUMBER}}/$BUILD_NUMBER/g" | kubectl apply -f -'
          bat 'kubectl apply -f service.yaml'
        }
      }
  }
}
post {
    success {
      slackSend(message: "Pipeline is successfully completed.")
    }
    failure {
      slackSend(message: "Pipeline failed. Please check the logs.")
    }
}
}
