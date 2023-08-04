### Jenkinsfile for quick task
========================================

pipeline {
    agent any

  stages {
  
  stage('Initial cleanup') {
    steps {
      dir("$WORKSPACE") {
        deleteDir()
      }
    }
  }

    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }

    stage('Test') {
      steps {
        script {
          sh 'echo "Testing Stage"'
        }
      }
    }
    

    stage('Package') {
      steps {
        script {
          sh 'echo "Packagin App"'
        }
      }
    }

    stage('Deploy') {
      steps {
        script {
          sh 'echo "Deploy to Dev"'
        }
      }
    }

    stage('Cleanup') {
      steps {
        cleanWs()
      }
    }
  }
}
   
   ###