pipeline {
    agent {
        node {
            label 'maven'
        }
    }
environment {
    PATH = "/opt/apache-maven-3.9.4/bin:$PATH"
}
    stages {
        stage('build') {
            steps {
                echo "---------------------Build Started------------------------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "---------------------Build Ended------------------------"
                
            }
        }
        stage("test"){
            steps{
                echo "---------------------Unit Test Started------------------------"
                sh 'mvn surefire-report:report'
                echo "---------------------Unit Test Ended------------------------"
            }
        }
    stage('SonarQube analysis') {
       environment {
           scannerHome = tool 'sonar-scanner';
       }
       steps{
          withSonarQubeEnv('sonarqube-server') { // If you have configured more than one global server connection, you can specify its name
              sh "${scannerHome}/bin/sonar-scanner"
          }
       }
    }
}
}
