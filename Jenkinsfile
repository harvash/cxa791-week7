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
                    ./gradlew checkstyle
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
          sh """
                ./gradlew build
                ls -l ./build/libs
                mv ./build/libs/calulator-0.0.1-SNAPSHOT.jar /mnt/calculator_${env.BRANCH_NAME}.jar
                ls -l /mnt
            """
          }
        }
      }
    }  
    stage('create container image and push to docker hub') {
      stages {
        stage('Operations for master branch'){
          when {branch 'master'}
          steps{
            sh """
                echo 'FROM openjdk:8-jre' > Dockerfile
                echo 'COPY ./calculator_${env.BRANCH_NAME}.jar app.jar' >> Dockerfile
                echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
                cat Dockerfile

                cp /mnt/calculator_${env.BRANCH_NAME}.jar .
                pwd
                ls -l
                /kaniko/executor --context `pwd` --destination harvash/calculator:1.0
              """
          }
        }
        stage('Operations for feature branch'){
          when {branch 'feature'}
          steps{
            sh """
                echo 'FROM openjdk:8-jre' > Dockerfile
                echo 'COPY ./calculator_${env.BRANCH_NAME}.jar app.jar' >> Dockerfile
                echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
                cat Dockerfile

                cp /mnt/calculator_${env.BRANCH_NAME}.jar .
                pwd
                ls -l
                /kaniko/executor --context `pwd` --destination harvash/calculator:0.1
              """
          }
        }
      }
    }
  }
}