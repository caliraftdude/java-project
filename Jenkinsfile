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
        post {
          success {
            archiveArtifacts artifacts: 'dist/*.jar', fingerprint: true
          }
        } 
     }
     stage('deploy') {
       agent {
         label 'CentOS'
       }
       steps {
         sh "ssh -o StrictHostKeyChecking=no jenkins@ansible.f5labs.com 'mkdir -p /var/www/html/rectangles/all/${env.BRANCH_NAME}' "
         sh "scp -o StrictHostKeyChecking=no dist/rectangle_${env.BUILD_NUMBER}.jar jenkins@ansible.f5labs.com:/var/www/html/rectangles/all/${env.BRANCH_NAME}/"
       }
     }
     stage("Running on CentOS") {
       agent {
         label 'CentOS'
       }
       steps {
         sh "wget http://ansible.f5labs.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.BUILD_NUMBER}.jar"
         sh "java -jar rectangle_${env.BUILD_NUMBER}.jar 3 4"
       }
     }
     stage("Runnning on Debian") {
       agent {
         docker 'openjdk:8u181-jre'
       }
       steps {
         sh "wget http://ansible.f5labs.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.BUILD_NUMBER}.jar"
         sh "java -jar rectangle_${env.BUILD_NUMBER}.jar 3 4"
       }
     }
     stage("Promote to Green") {
       agent {
         label 'master'
       }
       when {
         branch 'master'
       }
       steps {
         sh "cp /var/www/html/rectangles/all/rectangle_${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/rectangle_${env.BUILD_NUMBER}.jar"
       }
     }
     stage("Promote Development Branch to Master") {
       agent {
         label 'master'
       }
       when {
         branch 'development'
       }
       steps {
         echo "Stashing any local changes"
         sh 'git stash'
         echo "Checking out development branch"
         sh 'git checkout development'
         echo "Checking out master branch"
         sh 'git checkout master'
         echo "Merging development into master branch"
         sh 'git merge -m "Automated code merge from Jenkins - which should work now" development'
         echo "Pushing to origin master"
         sh 'git push origin master'
       }
     }
   }
}
