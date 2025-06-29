pipeline{
  agent any
  environment{
    AWS_REGION = 'us-east-2'
    IMAGE_NAME = 'jenkins-test'
    REPO_NAME = 'jenkins'
    IMAGE_TAG = 'latest'
  }
  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/TheMrYesac/jenkins'
      }
    }
    stage('Login to ECR') {
      steps {
        withCredentials([aws(credentialsId: 'aws', accesskeyVariable: 'AWS_ACCESS_KEY_ID', secretkeyVariable: 'AWS_SECRET_ACCESS_KEY')])
        withAWS(region: "${env.AWS_REGION}", credentials: 'aws') {
          powershell """
          (aws ecr get-login-password --region ${env:AWS_REGION} | docker login --username AWS --password-stdin 520320208152.dkr.ecr.${env.AWS_REGION}.amazonaws.com)
          """
        }
      }
    }
    stage('Build Docker Image') {
      steps {
        powershell """
        docker build -t $env.IMAGE_NAME:${env.IMAGE_TAG} .
        docker tag ${env.IMAGE_NAME}:${env.IMAGE_TAG} 520320208152.dkr.ecr.${env.AWS_REGION}.amazonaws.com/${env.REPO_NAME}:${env.IMAGE_TAG}
        """
      }
    }
    stage('Push to ECR') {
      steps {
        powershell """
        docker push 520320208152.dkr.ecr.${env.AWS_REGION}.amazonaws.com/${env.REPO_NAME}:${env.IMAGE_TAG}
        """
      }
    }
  }
}
