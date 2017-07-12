properties properties: [
  [$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator', artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '30', numToKeepStr: '10']],
  disableConcurrentBuilds()
]

timeout(60) {
  node {
    def buildNumber = env.BUILD_NUMBER
    def branchName = env.BRANCH_NAME
    def workspace = env.WORKSPACE
    def buildUrl = env.BUILD_URL

    def JAVA_JDK_8='/opt/jdk1.8.0_131'
    echo "Testing with Java $JAVA_JDK_8"
    
    def MAVEN_3='/opt/apache-maven-3.3.9'
    echo "Testing with Maven $MAVEN_3"

    withEnv(["Path+JDK=$JAVA_JDK_8/bin","Path+MAVEN=$MAVEN_3/bin","JAVA_HOME=$JAVA_JDK_8"]) {
      // PRINT ENVIRONMENT TO JOB
      echo "workspace directory is $workspace"
      echo "build URL is $buildUrl"
      echo "build Number is $buildNumber"
      echo "branch name is $branchName"
      echo "PATH is $env.PATH"
      sh 'java -version'
      sh 'mvn -version'

      try {
        stage('Clean workspace') {
          deleteDir()
        }

        stage('Checkout') {
          checkout scm
        }

        stage('Build') {
          sh "mvn clean package"
        }

        stage('Unit-Tests') {
          sh "mvn test -Dmaven.test.failure.ignore"
          step([
            $class     : 'JUnitResultArchiver',
            testResults: 'angular-spring-boot-webapp/target/surefire-reports/TEST*.xml'
          ])
        }

        stage('Integration-Tests') {
          node {
            // env.JAVA_HOME = '/Library/Java/JavaVirtualMachines/jdk1.8.0_25.jdk/Contents/Home/jre'
            
            checkout scm
            sh "mvn -Pdocker -Ddocker.host=tcp://127.0.0.1:2375 clean verify -Dmaven.test.failure.ignore"
            step([
              $class     : 'ArtifactArchiver',
              artifacts  : '**/target/*.jar',
              fingerprint: true
            ])
            step([
              $class     : 'JUnitResultArchiver',
              testResults: 'angular-spring-boot-webapp/target/failsafe-reports/TEST*.xml'
            ])
            publishHTML(target: [
              reportDir            : 'angular-spring-boot-webapp/target/site/serenity/',
              reportFiles          : 'index.html',
              reportName           : 'Serenity Test Report',
              keepAll              : true,
              alwaysLinkToLastBuild: true,
              allowMissing         : false
            ])
          }
        }

      } catch (e) {
        rocketSend channel: 'holi-demos', emoji: ':rotating_light:', message: 'Fehler'
        throw e
      }
    }
  }
}
