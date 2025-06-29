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
          # Ensure AWS credentials are explicitly set as PowerShell environment variables
          # Jenkins' withCredentials usually does this, but let's be explicit if there's a conflict.
          # The actual values come from the 'aws' credential in Jenkins, masked here for security.
          \$env:AWS_ACCESS_KEY_ID = \$env:AWS_ACCESS_KEY_ID
          \$env:AWS_SECRET_ACCESS_KEY = \$env:AWS_SECRET_ACCESS_KEY
          \$env:AWS_DEFAULT_REGION = '${env.AWS_REGION}' # Also set default region if needed by AWS CLI directly

          # Clear other potential AWS config environment variables to avoid conflicts
          Remove-Item ENV:AWS_SESSION_TOKEN -ErrorAction SilentlyContinue
          Remove-Item ENV:AWS_PROFILE -ErrorAction SilentlyContinue

          # Add verbose debugging for AWS CLI
          \$env:AWS_CLI_DEBUG = "1" # Set AWS CLI debug mode

          # Now proceed with getting the ECR password
          \$ECR_PASSWORD = aws ecr get-login-password --region ${env.AWS_REGION}
          
          # Unset debug mode immediately after for cleaner logs later
          Remove-Item ENV:AWS_CLI_DEBUG -ErrorAction SilentlyContinue

          if ([string]::IsNullOrEmpty(\$ECR_PASSWORD)) {
              Write-Host "ERROR: ECR password was EMPTY. Check previous AWS CLI output for errors."
              throw "Failed to retrieve ECR password."
          }
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
