pipeline {
  agent any
  options {
    skipDefaultCheckout(true)
  }
  environment {
    VERSION = "1.0.${sh(returnStdout: true, script: 'expr $BUILD_NUMBER - 67')}"
  }
  stages {
    stage('cleanup') {
      steps {
        cleanWs()
        checkout scm
      }
    }
    stage('dockerize') {
      when {
        allOf {
          branch 'dev';
          changeset 'website/*'
        }
      }
      tools {
        dockerTool '19.3'
      }
      steps {
        dir('website') {
          withCredentials([usernamePassword(credentialsId: 'docker-account', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
            sh 'echo $PASS | docker login -u $USER --password-stdin'
          }
          sh '''
            docker build --no-cache --tag remigiuszdonczyk/final-project .
            docker tag remigiuszdonczyk/final-project remigiuszdonczyk/final-project:$VERSION
            docker push remigiuszdonczyk/final-project:$VERSION
            docker push remigiuszdonczyk/final-project
            docker image rm remigiuszdonczyk/final-project:$VERSION
            docker image rm remigiuszdonczyk/final-project
            docker logout
          '''
        }
      }
    }
    stage('terraform') {
      when { branch 'dev' }
      tools {
        terraform '1.2.5'
      }
      environment {
        AWS_ACCESS_KEY_ID = credentials('aws-access')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret')
      }
      steps {
        sh '''
          terraform init
          terraform plan -out .plan
        '''
        retry(2) {
          sh 'terraform apply .plan'
        }
      }
    }
    stage('test') {
      when { branch 'dev' }
      steps {
        sh 'ls'
      }
    }
    stage('extinction') {
      when { branch 'dev' }
      tools {
        terraform '1.2.5'
      }
      environment {
        AWS_ACCESS_KEY_ID = credentials('aws-access')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret')
      }
      steps {
        input message: 'Do you wish to perform extinction?', ok: 'Approve'
        sh '''
          terraform init
          terraform plan -destroy -out .plan
        '''
        retry(2) {
          sh 'terraform apply .plan'
        }
      }
    }
  }
}

