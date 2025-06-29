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
          Remove-Item ENV:AWS_ACCESS_KEY_ID -ErrorAction SilentlyContinue
          Remove-Item ENV:AWS_SECRET_ACCESS_KEY -ErrorAction SilentlyContinue
          Remove-Item ENV:AWS_SESSION_TOKEN -ErrorAction SilentlyContinue
          Remove-Item ENV:AWS_DEFAULT_REGION -ErrorAction SilentlyContinue
          Remove-Item ENV:AWS_PROFILE -ErrorAction SilentlyContinue

          \$env:AWS_CLI_DEBUG = "1" # Set AWS CLI debug mode

          \$ECR_PASSWORD = aws ecr get-login-password --region ${env.AWS_REGION}
          Remove-Item ENV:AWS_CLI_DEBUG -ErrorAction SilentlyContinue # Unset debug mode

          if ([string]::IsNullOrEmpty(\$ECR_PASSWORD)) {
              Write-Host "ERROR: ECR password was EMPTY. Check previous AWS CLI output for errors."
              throw "Failed to retrieve ECR password."
          }
          Write-Host "Length of ECR_PASSWORD: " + \$ECR_PASSWORD.Length
          Write-Host "First 10 chars of ECR_PASSWORD: " + \$ECR_PASSWORD.Substring(0,10)

          \$ECR_REGISTRY_URL = "520320208152.dkr.ecr.\$(\$env:AWS_REGION).amazonaws.com"
          Write-Host "DEBUG: ECR_REGISTRY_URL is '\$ECR_REGISTRY_URL'"

          # ***** CRITICAL CHANGE HERE: Use Start-Process for robust piping *****
          # This should pass the token more reliably to Docker's stdin.
          # We'll also capture stderr/stdout for more detailed output if it fails.
          \$dockerLoginCommand = "docker"
          \$dockerLoginArgs = @(
              "login",
              "--username", "AWS",
              "--password-stdin",
              "\$ECR_REGISTRY_URL"
          )

          # Prepare StandardInput as a MemoryStream containing the password
          \$inputStream = [System.IO.MemoryStream]::new((New-Object System.Text.ASCIIEncoding).GetBytes(\$ECR_PASSWORD))

          # Invoke Start-Process and capture its output
          \$process = Start-Process -FilePath \$dockerLoginCommand -ArgumentList \$dockerLoginArgs -NoNewWindow -Wait -PassThru -RedirectStandardOutput -RedirectStandardError -StandardInput \$inputStream
          \$stdOut = \$process.StandardOutput.ReadToEnd()
          \$stdErr = \$process.StandardError.ReadToEnd()
          \$inputStream.Dispose() # Clean up the stream

          Write-Host "---- Docker Login STDOUT ----"
          Write-Host \$stdOut
          Write-Host "-----------------------------"
          Write-Host "---- Docker Login STDERR ----"
          Write-Host \$stdErr
          Write-Host "-----------------------------"

          if (\$process.ExitCode -ne 0) {
              throw "Docker login failed with exit code \$(\$process.ExitCode). See stderr/stdout above."
          } else {
              Write-Host "Docker login SUCCEEDED!"
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
