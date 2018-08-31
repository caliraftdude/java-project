 pipeline {
   agent none

   options {
      buildDiscarder(logRotator(numToKeepStr: '2',artifactNumToKeepStr: '1'))
   }

   stages {
     stage('Unit Tests') {
       agent {
         label 'CentOS'
       }
       steps {
         sh 'ant -f test.xml -v'
         junit 'reports/result.xml'
       }
     }
     stage('build') {
        agent {
          label 'CentOS'
        }
        steps {
        sh 'ant -f build.xml -v'
        } 
     }
     stage('deploy') {
       agent {
         label 'CentOS'
       }
       steps {
         sh "scp -v -o StrictHostKeyChecking=no dist/rectangle_${env.BUILD_NUMBER}.jar jenkins@ansible.f5labs.com:/var/www/html/rectangles/all/"
       }
     }
     stage("Running on CentOS") {
       agent {
         label 'CentOS'
       }
       steps {
         sh "wget http://ansible.f5labs.com/rectangles/all/rectangle_${env.BUILD_NUMBER}.jar"
         sh "java -jar rectangle_${env.BUILD_NUMBER}.jar 3 4"
       }
     }
   }  
   post {
      always {
         archiveArtifacts artifacts: 'dist/*.jar', fingerprint: true
      }
   }
}
