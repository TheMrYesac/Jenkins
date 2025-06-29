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
        withCredentials([aws(credentialsId: 'aws')]) {
          powershell """
          \$ECR_PASSWORD = aws ecr get-login-password --region ${env.AWS_REGION}
          Write-Host "Length of ECR_PASSWORD: \$(\$ECR_PASSWORD.Length)"
          Write-Host "First 10 chars of ECR_PASSWORD: \$(\$ECR_PASSWORD.Substring(0,10))" # For debugging only, remove in production!

          # This line remains correct for its assignment
          \$ECR_REGISTRY_URL = "520320208152.dkr.ecr.${env.AWS_REGION}.amazonaws.com"
          Write-Host "DEBUG: ECR_REGISTRY_URL is '\$ECR_REGISTRY_URL'" # Add this for further debugging

          # FIX IS HERE: Use the fully formed URL directly, or ensure $ECR_REGISTRY_URL is used correctly
          echo \$ECR_PASSWORD | docker login --username AWS --password-stdin \$ECR_REGISTRY_URL
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
