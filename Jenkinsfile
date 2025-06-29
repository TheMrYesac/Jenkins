pipeline {
  agent any
  environment {
    AWS_REGION = 'us-east-2'
    IMAGE_NAME = 'jenkins-test'
    REPO_NAME = 'jenkins'
    IMAGE_TAG = 'latest'
    // Define the ECR_REGISTRY here as a Jenkins environment variable,
    // ensuring Groovy interpolates it fully at pipeline start.
    // NOTE: We don't need 'env.' when referring to other environment variables
    // within the 'environment' block itself.
    ECR_REGISTRY_URL_FULL = "520320208152.dkr.ecr.${AWS_REGION}.amazonaws.com"
  }
  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://Github.com/TheMrYesac/jenkins'
      }
    }
    stage('Login to ECR') {
      steps {
        withCredentials([aws(credentialsId: 'aws')]) {
          powershell """
          # These variables are PowerShell variables, correctly escaped
          \$ECR_PASSWORD = aws ecr get-login-password --region ${env.AWS_REGION}
          Write-Host "Length of ECR_PASSWORD: \$(\$ECR_PASSWORD.Length)"
          Write-Host "First 10 chars of ECR_PASSWORD: \$(\$ECR_PASSWORD.Substring(0,10))" # For debugging only, remove in production!

          # IMPORTANT: Use the Jenkins environment variable directly, letting Groovy interpolate it.
          # The variable \$ECR_REGISTRY_URL inside PowerShell is redundant now if you defined it globally.
          Write-Host "DEBUG: ECR_REGISTRY_URL_FULL is '${env.ECR_REGISTRY_URL_FULL}'"

          echo \$ECR_PASSWORD | docker login --username AWS --password-stdin ${env.ECR_REGISTRY_URL_FULL}
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
        docker tag ${env.IMAGE_NAME}:${env.IMAGE_TAG} ${env.ECR_REGISTRY_URL_FULL}/${env.REPO_NAME}:${env.IMAGE_TAG}
        """
      }
    }
    stage('Push to ECR') {
      steps {
        powershell """
        docker push ${env.ECR_REGISTRY_URL_FULL}/${env.REPO_NAME}:${env.IMAGE_TAG}
        """
      }
    }
  }
}
