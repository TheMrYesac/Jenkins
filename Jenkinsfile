pipeline{
  agent any
  environment{
    VENV = 'venv'
  }
  stages{
    stage('Checkout git'){
      steps{
        git branch: 'main', url: 'https://github.com/TheMrYesac/jenkins'
      }
    }
    stage('set up the venv'){
      steps{
        sh 'python -m venv %VENV%'
        sh '%VENV%\\Scripts\\python -m pip install --upgrade pip'
        sh '%VENV%\\Scripts\\pip install -r requirements.txt'
      }
    }
    stage('RUN THE TESTS'){
      steps{
        sh '%VENV%\\Scripts\\python -m unittest discover -s tests'
      }
    }
  }
}
