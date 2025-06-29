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
          // Using triple-single quotes to prevent Groovy interpolation within the block
          // Jenkins environment variables are injected using GString interpolation outside the block.
          powershell '''
          # Set AWS CLI debug mode FIRST. This is crucial for debugging.
          $env:AWS_CLI_DEBUG = "1"

          # Clear *other* AWS-related environment variables that might confuse aws cli,
          # but *not* the primary AWS_ACCESS_KEY_ID/SECRET_ACCESS_KEY set by withCredentials.
          Remove-Item ENV:AWS_SESSION_TOKEN -ErrorAction SilentlyContinue
          Remove-Item ENV:AWS_PROFILE -ErrorAction SilentlyContinue
          Remove-Item ENV:AWS_SHARED_CREDENTIALS_FILE -ErrorAction SilentlyContinue

          # Now proceed with getting the ECR password.
          # The verbose AWS CLI debug output should appear here.
          $ECR_PASSWORD = aws ecr get-login-password --region ''' + env.AWS_REGION + '''

          # Unset debug mode immediately after for cleaner logs later
          Remove-Item ENV:AWS_CLI_DEBUG -ErrorAction SilentlyContinue

          if ([string]::IsNullOrEmpty($ECR_PASSWORD)) {
              Write-Host "--- AWS CLI Output (if any) ---"
              throw "Failed to retrieve ECR password. AWS CLI reported: Unable to locate credentials."
          }

          Write-Host "Length of ECR_PASSWORD: " + $ECR_PASSWORD.Length
          Write-Host "First 10 chars of ${ECR_PASSWORD}: " + $ECR_PASSWORD.Substring(0,10)

          $ECR_REGISTRY_URL = "520320208152.dkr.ecr.$($env:AWS_REGION).amazonaws.com"
          Write-Host "DEBUG: ECR_REGISTRY_URL is '$ECR_REGISTRY_URL'"

          # ***** Docker Login: Using a temporary file for robust password piping *****
          $tempPasswordFile = Join-Path $env:TEMP "docker_ecr_pass.txt"
          Set-Content -Path $tempPasswordFile -Value $ECR_PASSWORD -Encoding ASCII

          # Construct the command to be executed by cmd.exe (which handles 'type |')
          # FIX: Doubled the quotes for inner string
          $command = "type ""$tempPasswordFile"" | docker login --username AWS --password-stdin ""$ECR_REGISTRY_URL"""

          # Removed -RedirectStandardOutput and -RedirectStandardError as they are not needed for ReadToEnd()
          $process = Start-Process -FilePath "cmd.exe" -ArgumentList "/c", $command -NoNewWindow -Wait -PassThru
          $stdOut = $process.StandardOutput.ReadToEnd()
          $stdErr = $process.StandardError.ReadToEnd()

          Remove-Item $tempPasswordFile -ErrorAction SilentlyContinue

          Write-Host "---- Docker Login STDOUT (via temp file) ----"
          Write-Host $stdOut
          Write-Host "------------------------------------------"
          Write-Host "---- Docker Login STDERR (via temp file) ----"
          Write-Host $stdErr
          Write-Host "------------------------------------------"

          if ($process.ExitCode -ne 0) {
              throw "Docker login failed with exit code $($process.ExitCode). See stderr/stdout above."
          } else {
              Write-Host "Docker login SUCCEEDED!"
          }
          '''
        }
      }
    }
    stage('Build Docker Image') {
      steps {
        powershell '''
        $ECR_REGISTRY_URL = "520320208152.dkr.ecr.''' + env.AWS_REGION + '''.amazonaws.com"
        docker build -t ''' + env.IMAGE_NAME + ''':''' + env.IMAGE_TAG + ''' .
        docker tag ''' + env.IMAGE_NAME + ''':''' + env.IMAGE_TAG + ''' $ECR_REGISTRY_URL/''' + env.REPO_NAME + ''':''' + env.IMAGE_TAG + '''
        '''
      }
    }
    stage('Push to ECR') {
      steps {
        powershell '''
        $ECR_REGISTRY_URL = "520320208152.dkr.ecr.''' + env.AWS_REGION + '''.amazonaws.com"
        docker push $ECR_REGISTRY_URL/''' + env.REPO_NAME + ''':''' + env.IMAGE_TAG + '''
        '''
      }
    }
  }
}
