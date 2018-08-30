 pipeline {
   agent {
      label 'CentOS'
   }
   stages {
    stage('build') {
      steps {
        sh 'ant -f build.xml -v'
      }
    }
  }
}
