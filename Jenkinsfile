pipeline{
  agent any
  environment{
    AWS_REGION = 'us-east-2'
    IMAGE_NAME = 'jenkins-test'
    REPO_NAME = 'jenkins'
  }
  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/TheMrYesac/jenkins'
      }
    }
    stage('Tag the Image') {
      steps {
        script{
          IMAGE_TAG = 'latest'
        }
      }
    }
    stage('Login to ECR') {
      steps {
        withAWS(region: "${env.AWS_REGION}", credentials: 'aws-creds') {
          powershell '''
          $ecrLogin = aws ecr get-login-password --region ${env.AWS_REGION}

          docker login --username AWS --password-stdin $ecrLogin https://520320208152.dkr.ecr.us-east-2.amazonaws.com
          '''
        }
      }
    }
    stage('Build Docker Image') {
      steps {
        powershell '''
        docker build -t $env.IMAGE_NAME:${env.IMAGE_TAG} .
        docker tag ${env.IMAGE_NAME}:${env.IMAGE_TAG} 520320208152.dkr.ecr.us-east-2.amazonaws.com/jenkins:latest
        '''
      }
    }
    stage('Push to ECR') {
      steps {
        powershell '''
        docker push 520320208152.dkr.ecr.us-east-2.amazonaws.com/jenkins:latest
        '''
      }
    }
  }
}
