 pipeline {
   agent {
      label 'CentOS'
   }

   options {
      buildDiscarder(logRotator(numToKeepStr: '2',artifactNumToKeppStr: '1'))
   }

   stages {
     stage('build') {
        steps {
        sh 'ant -f build.xml -v'
        } 
     }
   }
   post {
      always {
         archiveArtifacts artifacts: 'dist/*.jar', fingerprint: true
      }
   }
}
