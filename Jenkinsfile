pipeline {
  agent {
    kubernetes {
      yamlFile 'agents.yaml'
    }
  }
  stages {
    stage('Checkout SCM') {
      steps {
        git branch: env.BRANCH_NAME, url:'https://github.com/harvash/cxa791-week7.git'
      }
    }
    stage('Test & Build Java code'){
      stages {
        stage ('Run tests for master branch') { 
          when { branch 'master' }
          steps { 
            container('gradle') {
              sh '''cd Chapter08/sample1
                    chmod +x gradlew      
                    ./gradlew test
                    ./gradlew jacocoTestCoverageVerification
                    ./gradlew jacocoTestReport
                    ./gradlew checkstyleMain
                ''' 
              publishHTML (
                target: [
                  reportDir: 'Chapter08/sample1/build/reports/jacoco/test/html/',
                  reportFiles: 'index.html',
                  reportName: 'JaCoCo Report'
                ]
              )
              publishHTML (
                target: [
                  reportDir: 'Chapter08/sample1/build/reports/checkstyle/',
                  reportFiles: 'master.html',
                  reportName: 'CheckStyle Report'
                ]
              )

            }
          }
        }
        stage('Build jar') {
          steps {
            container('gradle') {
              sh """
                    cd Chapter08/sample1
                    ls -l
                    ./gradlew build
                    ls -l ./build/libs
                    mv ./build/libs/calculator-0.0.1-SNAPSHOT.jar /mnt                       
                    ls -l /mnt
                """
            }
          }
        }
      }
    }  
    stage('create container image and push to docker hub') {
      stages {
        stage('build Dockerfile'){
          steps{
            container('kaniko') {
              sh """
                  echo 'FROM openjdk:8-jre' > Dockerfile
                  echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar  app.jar' >> Dockerfile
                  echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
                  cat Dockerfile
                  ls -l /mnt
                  cp /mnt/calculator-0.0.1-SNAPSHOT.jar  .
                  pwd
                  ls -l
                  
                """
            }
          }
        }
        stage('Operations for master branch'){
          steps{
            container('kaniko') {
              sh  """
                  /kaniko/executor --context `pwd` --destination harvash/calculator:1.0
                  """
            }
          }
        }
        stage('Operations for feature branch'){
          steps{
            container('kaniko') {
              sh  """
                  /kaniko/executor --context `pwd` --destination harvash/calculator:0.1
                  """
            }
          }
        }
      }
    }
  }
}