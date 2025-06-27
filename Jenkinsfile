pipeline{
  agent any
  environment{
    VENV = 'venv'
  }
  stages{
    stage('Checkout git'){
      steps{
        git branch: 'main', url: 'https://github.com/Parth2k3/test-flask'
      }
    }
    stage('set up the venv'){
      steps{
        bat 'python -m venv %VENV%'
        bat '%VENV%\\Scripts\\python -m pip install --upgrade pip'
        bat '%VENV%\\Scripts\\pip install -r requirements.txt'
      }
    }
    stage('RUN THE TESTS'){
      steps{
        bat '%VENV%\\Scripts\\python -m unittest discover -s tests'
      }
    }
  }
}
