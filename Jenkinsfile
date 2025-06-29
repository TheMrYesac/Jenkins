pipeline{
  agent any
  environment{
    VENV = 'venv'
  }
  stages{
    stage('Checkout'){
      steps{
        git branch: 'main', url: 'https://github.com/TheMrYesac/jenkins'
      }
    }
    stage('Set up VENV'){
      steps{
        bat 'python -m venv %%VENV%%'
        bat '%VENV%\\Scripts\\python -m pip install --upgrade pip'
        bat '%VENV%\\Scripts\\pip install -r requirements.txt'
      }
    }
    stage('Run Tests'){
      steps{
        bat '%VENV%\\Scripts\\python -m unittest discover -s tests'
      }
    }
  }
}
