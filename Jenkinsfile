 pipeline {
   agent none

   options {
      buildDiscarder(logRotator(numToKeepStr: '2',artifactNumToKeepStr: '1'))
   }

   environment {
      MAJOR_VERSION = 1
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
         sh "scp -o StrictHostKeyChecking=no dist/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar jenkins@ansible.f5labs.com:/var/www/html/rectangles/all/${env.BRANCH_NAME}/"
       }
     }
     stage("Running on CentOS") {
       agent {
         label 'CentOS'
       }
       steps {
         sh "wget http://ansible.f5labs.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
         sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
       }
     }
     stage("Runnning on Debian") {
       agent {
         docker 'openjdk:8u181-jre'
       }
       steps {
         sh "wget http://ansible.f5labs.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
         sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
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
         sh "cp /var/www/html/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
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
         sh 'git pull origin'
         sh 'git checkout master'
         echo "Merging development into master branch"
         sh 'git merge -m "Automated code merge from Jenkins - final test" development'
         echo "Pushing to origin master"
         sh 'git push origin master'
         echo "Tagging the Release"
         sh "git tag rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
         sh "git push origin rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
       }
       post {
          success {
             emailext(
                subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Development Promoted to Master",
                body: """<p>'${env.JOB_NAME} [${env.BUILD_NUMBER}]' Development Promoted to Master":</p>
                <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
                to: "d.holland@f5.com"
             )
          }
       }
     }
     post {
        failure {
           emailext(
              subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Failed!",
              body: """<p>'${env.JOB_NAME} [${env.BUILD_NUMBER}]' Failed!":</p>
              <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
              to: "d.holland@f5.com"
           )
        }
     }
   }
}
