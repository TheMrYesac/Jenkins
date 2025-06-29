pipeline{
  agent any
  environment{
    AWS_REGION = 'us-east-2'
    IMAGE_NAME = 'jenkins-test'
    REPO_NAME = 'jenkins'
    IMAGE_TAG = 'latest'
    // Define the ECR_REGISTRY here as a Jenkins environment variable
    ECR_REGISTRY = "520320208152.dkr.ecr.${AWS_REGION}.amazonaws.com" // <--- Moved here
  }
  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/TheMrYesac/jenkins'
      }
    }
    stage('Login to ECR') {
      steps {
        withCredentials([aws(credentialsId: 'aws')]) {
          powershell """
          \$ECR_PASSWORD = aws ecr get-login-password --region ${env.AWS_REGION}
          Write-Host "Length of ECR_PASSWORD: \$(\$ECR_PASSWORD.Length)"
          Write-Host "First 10 chars of ECR_PASSWORD: \$(\$ECR_PASSWORD.Substring(0,10))" # For debugging only, remove in production!

          # Access ECR_REGISTRY as a Jenkins environment variable directly within PowerShell
          echo \$ECR_PASSWORD | docker login --username AWS --password-stdin ${env.ECR_REGISTRY}
          if (\$LASTEXITCODE -ne 0) {
              throw "Docker login failed with exit code \$LASTEXITCODE"
          }
          """
        }
      }
    }
    stage('Build Docker Image') {
      steps {
        powershell """
        docker build -t ${env.IMAGE_NAME}:${env.IMAGE_TAG} .
        docker tag ${env.IMAGE_NAME}:${env.IMAGE_TAG} ${env.ECR_REGISTRY}/${env.REPO_NAME}:${env.IMAGE_TAG}
        """
      }
    }
    stage('Push to ECR') {
      steps {
        powershell """
        docker push ${env.ECR_REGISTRY}/${env.REPO_NAME}:${env.IMAGE_TAG}
        """
      }
    }
  }
}
