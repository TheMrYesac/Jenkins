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
          # Set AWS CLI debug mode FIRST. This is crucial for debugging.
          \$env:AWS_CLI_DEBUG = "1"

          # Clear *other* AWS-related environment variables that might confuse aws cli,
          # but *not* the primary AWS_ACCESS_KEY_ID/SECRET_ACCESS_KEY set by withCredentials.
          Remove-Item ENV:AWS_SESSION_TOKEN -ErrorAction SilentlyContinue
          Remove-Item ENV:AWS_PROFILE -ErrorAction SilentlyContinue
          # Also, ensure AWS_SHARED_CREDENTIALS_FILE isn't pointing somewhere weird if you have one.
          Remove-Item ENV:AWS_SHARED_CREDENTIALS_FILE -ErrorAction SilentlyContinue


          # Now proceed with getting the ECR password.
          # The verbose AWS CLI debug output should appear here.
          \$ECR_PASSWORD = aws ecr get-login-password --region ${env.AWS_REGION}

          # Unset debug mode immediately after for cleaner logs later
          Remove-Item ENV:AWS_CLI_DEBUG -ErrorAction SilentlyContinue

          if ([string]::IsNullOrEmpty(\$ECR_PASSWORD)) {
              Write-Host "--- AWS CLI Output (if any) ---"
              # We expect debug output here. If not, the issue is before AWS CLI logs anything.
              # Throw an error to halt the pipeline.
              throw "Failed to retrieve ECR password. AWS CLI reported: Unable to locate credentials."
          }

          Write-Host "Length of ECR_PASSWORD: " + \$ECR_PASSWORD.Length
          Write-Host "First 10 chars of ECR_PASSWORD: " + \$ECR_PASSWORD.Substring(0,10)

          \$ECR_REGISTRY_URL = "520320208152.dkr.ecr.\$(\$env:AWS_REGION).amazonaws.com"
          Write-Host "DEBUG: ECR_REGISTRY_URL is '\$ECR_REGISTRY_URL'"

          # ***** Docker Login (using Start-Process from last attempt) *****
          \$dockerLoginCommand = "docker"
          \$dockerLoginArgs = @(
              "login",
              "--username", "AWS",
              "--password-stdin",
              "\$ECR_REGISTRY_URL"
          )

          \$inputStream = [System.IO.MemoryStream]::new((New-Object System.Text.ASCIIEncoding).GetBytes(\$ECR_PASSWORD))

          \$process = Start-Process -FilePath \$dockerLoginCommand -ArgumentList \$dockerLoginArgs -NoNewWindow -Wait -PassThru -RedirectStandardOutput -RedirectStandardError -StandardInput \$inputStream
          \$stdOut = \$process.StandardOutput.ReadToEnd()
          \$stdErr = \$process.StandardError.ReadToEnd()
          \$inputStream.Dispose()

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
