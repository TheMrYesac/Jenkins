pipeline {
  agent any
  environment {
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
          # REMOVED: Remove-Item "\$HOME/.aws" -Recurse -Force -ErrorAction SilentlyContinue
          # We manually configured this with psexec, so we don't want to delete it.

          # These environment variable removals are generally safe, as they just clear dynamic vars.
          Remove-Item ENV:AWS_ACCESS_KEY_ID -ErrorAction SilentlyContinue
          Remove-Item ENV:AWS_SECRET_ACCESS_KEY -ErrorAction SilentlyContinue
          Remove-Item ENV:AWS_SESSION_TOKEN -ErrorAction SilentlyContinue
          Remove-Item ENV:AWS_DEFAULT_REGION -ErrorAction SilentlyContinue
          Remove-Item ENV:AWS_PROFILE -ErrorAction SilentlyContinue

          # Now proceed with getting the ECR password, which should use the credentials from withCredentials (or the configured system profile)
          \$ECR_PASSWORD = aws ecr get-login-password --region ${env.AWS_REGION}
          Write-Host "Length of ECR_PASSWORD: " + \$ECR_PASSWORD.Length
          Write-Host "First 10 chars of ECR_PASSWORD: " + \$ECR_PASSWORD.Substring(0,10) # For debugging only, remove in production!

          \$ECR_REGISTRY_URL = "520320208152.dkr.ecr.\$(\$env:AWS_REGION).amazonaws.com"
          Write-Host "DEBUG: ECR_REGISTRY_URL is '\$ECR_REGISTRY_URL'"

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
        \$ECR_REGISTRY_URL = "520320208152.dkr.ecr.\$(\$env:AWS_REGION).amazonaws.com"
        docker build -t ${env.IMAGE_NAME}:${env.IMAGE_TAG} .
        docker tag ${env.IMAGE_NAME}:${env.IMAGE_TAG} \$ECR_REGISTRY_URL/${env.REPO_NAME}:${env.IMAGE_TAG}
        """
      }
    }
    stage('Push to ECR') {
      steps {
        powershell """
        \$ECR_REGISTRY_URL = "520320208152.dkr.ecr.\$(\$env:AWS_REGION).amazonaws.com"
        docker push \$ECR_REGISTRY_URL/${env.REPO_NAME}:${env.IMAGE_TAG}
        """
      }
    }
  }
}
