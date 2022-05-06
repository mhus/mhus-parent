pipeline {
  agent {
    kubernetes {
      yamlFile 'jenkins-pod.yaml'
    }
  }
  stages {
    stage('Run maven') {
      steps {
        container('maven') {
          sh 'mvn package'
        }
      }
    }
  }
}