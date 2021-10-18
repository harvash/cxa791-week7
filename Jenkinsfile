pipeline {
  agent {
    kubernetes {
      yamlFile 'agents.yaml'
    }
  }
  stages {
    stage('Test & Build Calculator Java') {
      stage('Checkout SCM') {
        steps {
          git branch: env.BRANCH_NAME, url:'https://github.com/harvash/cxa791-week7.git'
        }
      }
      stage ('Run tests for master branch') { 
        when { branch: master }
        steps { 
          container('gradle') {
            sh '''cd Chapter08/sample1
                  chmod +x gradlew      
                  ./gradlew test
                  ./gradlew jacocoTestCoverageVerification
                  ./gradlew jacocoTestReport
                  ./gradlew checkstylemaster
              ''' 
            publishHTML (
              target: [
                reportDir: 'Chapter08/sample1/build/reports/jacoco/test/html',
                reportFiles: 'index.html',
                reportName: 'JaCoCo Report'
              ]
            )
            publishHTML (
              target: [
                reportDir: 'Chapter08/sample1/build/reports/checkstyle',
                reportFiles: 'master.html',
                reportName: 'CheckStyle Report'
              ]
            )
            try {
              sh '''./gradlew test'''
            } catch(all) {
                echo "Unit tests fail"
            }
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
    stage('Push image to Docker Hub'){
      when {branch: master ; branch: feature}
      steps {
        container('kaniko') {
          sh """echo 'FROM openjdk:8-jre' > Dockerfile
                echo 'COPY ./calculator_${env.BRANCH_NAME}.jar app.jar' >> Dockerfile
                echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
                cat Dockerfile
                ls -l /mnt
                cp /mnt/calculator_${env.BRANCH_NAME}.jar .
                pwd
                ls -al
                if ["${env.BRANCH_NAME}" == "master" ]; then
                  /kaniko/executor --context `pwd` --destination harvash/calculator:1.0
                else
                  /kaniko/executor --context `pwd` --destination harvash/calculator-feature:0.1
                fi
                """         
        }
      }
    }
  }
}