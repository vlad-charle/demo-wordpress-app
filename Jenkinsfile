def gv

pipeline {
  environment {
          AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
          AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
      }

  agent any

  stages {
    stage('Load Groovy script') {
      steps{
        script {
          gv = load "script.groovy"
        }
      }
    }
    stage('SonarQube') {
      environment {
        scannerHome = tool 'SonarQubeScanner'
      }
      steps {
        withSonarQubeEnv('sonarqube') {
          sh "${scannerHome}/bin/sonar-scanner"
        }

        timeout(time: 10, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
    stage('Build image') {
      steps{
        script {
          gv.buildImage()
        }
      }
    }
    stage('Push Image') {
      steps{
        script {
          gv.pushImage()
        }
      }
    }
    stage('Deploy App') {
      steps {
          withCredentials([file(credentialsId: 'mykubeconfig', variable: 'KUBECONFIG')]) {
            sh 'kubectl apply -f k8s'
        }
      }
    }
  }
  post {
    success {
        slackSend color: 'good', message: "Build succesful: ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
    }
    failure {
        slackSend color: 'danger', message: "Build failed: ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
    }
  }
}