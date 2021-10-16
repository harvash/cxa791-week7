podTemplate(yaml: '''
  apiVersion: v1 
  kind: Pod
  spec:
    containers:
    - name: gradle
      image: gradle:6.3-jdk14
      command:
      - sleep
      args:
      - 99d
      resources:
        limits:
          cpu: 500m
          memory: 500M
        requests:
          cpu: 100m
          memory: 256M
      volumeMounts:
      - name: shared-storage
        mountPath: /mnt
    - name: kaniko
      image: gcr.io/kaniko-project/executor:debug
      command:
      - sleep
      args:
      - 9999999
      volumeMounts:
      - name: shared-storage
        mountPath: /mnt
      - name: kaniko-secret
        mountPath: /kaniko/.docker
      resources:
        limits:
          cpu: 500m
          memory: 500M
        requests:
          cpu: 100m
          memory: 256M
    - name: calculator-feature
      image: harvash/calculator:0.1
      imagePullPolicy: Always
      ports:
      - containerPort: 8080
      resources:
        limits:
          cpu: 500m
          memory: 500M
        requests:
          cpu: 100m
          memory: 256M
    - name: calculator-master
      image: harvash/calculator:1.0
      imagePullPolicy: Always
      ports:
      - containerPort: 8080
      resources:
        limits:
          cpu: 500m
          memory: 500Mi
        requests:
          cpu: 100m
          memory: 256Mi
    restartPolicy: Never
    volumes:
    - name: shared-storage
      persistentVolumeClaim:
        claimName: kaniko-pv-claim
    - name: kaniko-secret
      secret:
        secretName: dockercred
        items:
        - key: .dockerconfigjson
          path: config.json'''
)  {
  node(POD_LABEL) {
    stage('Run pipeline against a gradle project') {
      git branch: env.BRANCH_NAME, url:'https://github.com/harvash/cxa791-week_6.git'
      container('gradle') {
        stage('Build a gradle project') {
          
          sh '''cd Chapter08/sample1
                chmod +x gradlew '''
          
          if (env.BRANCH_NAME == 'master') {
            sh '''cd Chapter08/sample1
                  ./gradlew test''' 
          }
        }
        stage('Code coverage') {
          if (env.BRANCH_NAME == 'master') {
            sh '''pwd
                  cd Chapter08/sample1
                  ./gradlew jacocoTestCoverageVerification
                  ./gradlew jacocoTestReport'''
            publishHTML (
              target: [
                reportDir: 'Chapter08/sample1/build/reports/jacoco/test/html',
                reportFiles: 'index.html',
                reportName: 'JaCoCo Report'
              ]
            )
          }
        }
        stage('Checkstyle') {
          if (env.BRANCH_NAME == 'feature') {
            try {
              sh '''pwd
                    cd Chapter08/sample1
                    ./gradlew checkstylemaster'''
            } catch(all) {
                echo "checkstyle fails"
            }
            publishHTML (
              target: [
                reportDir: 'Chapter08/sample1/build/reports/checkstyle',
                reportFiles: 'master.html',
                reportName: 'CheckStyle Report'
              ]
            )
          }
        }
        stage('Unit_test') {
          if (env.BRANCH_NAME == 'feature') {
            try {
              sh '''pwd
                    cd Chapter08/sample1
                    ./gradlew test'''
            } catch(all) {
                echo "Unit tests fail"
            }
          }
        }
        stage('Build jar') {
          sh """pwd
                cd Chapter08/sample1
                chmod +x gradlew
                ./gradlew build
                ls -l ./build/libs
                mv ./build/libs/calulator-0.0.1-SNAPSHOT.jar /mnt/calculator_${env.BRANCH_NAME}.jar
                ls -l /mnt"""
        }
      }
    }
    stage('Build Java Image') {
      container('kaniko') {
        stage('Build a gradle project') {
          if (env.BRANCH_NAME == 'feature' || env.BRANCH_NAME == 'master') {
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
    stage('Deploy Calculator jar') {
      container('calculator'){

      }
    }
  }
}