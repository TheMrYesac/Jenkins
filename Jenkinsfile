pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/TheMrYesac/Jenkins'
            }
        }
        stage('Build') {
            steps{
                sh 'echo "Building the app"'
            }
        }
        stage('Test') {
              steps{
                  sh 'echo "Running tests"'
              }
        }
        stage('Deploy') {
            steps {
                sh 'echo "Deploying app"'
            }
        }
    }
    post {
        success {
            sh 'echo "build successful"'
        }
        failure {
            sh 'echo "build failed"'
        }
    }
}
