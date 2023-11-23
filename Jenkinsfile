def registry = 'https://twittertrendjforgcicd.jfrog.io'
def imageName = 'twittertrendjforgcicd.jfrog.io/valaxy-docker/ttrend'
def version   = '2.0.2'
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

stage("Quality Gate"){
    steps {
        script {
        timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
            def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
            if (qg.status != 'OK') {
      error "Pipeline aborted due to quality gate failure: ${qg.status}"
    }
  }
}
    }
  }
stage("Jar Publish") {
        steps {
            script {
                    echo '<--------------- Jar Publish Started --------------->'
                     def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"cb52d53b-89d7-4192-b1e4-a625fed0979a"
                     def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                     def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "libs-release-local/{1}",
                              "flat": "false",
                              "props" : "${properties}",
                              "exclusions": [ "*.sha1", "*.md5"]
                            }
                         ]
                     }"""
                     def buildInfo = server.upload(uploadSpec)
                     buildInfo.env.collect()
                     server.publishBuildInfo(buildInfo)
                     echo '<--------------- Jar Publish Ended --------------->'  
            
            }
        }   
    }
stage(" Docker Build ") {
      steps {
        script {
           echo '<--------------- Docker Build Started --------------->'
           app = docker.build(imageName+":"+version)
           echo '<--------------- Docker Build Ends --------------->'
        }
      }
    }

    stage (" Docker Publish "){
        steps {
            script {
               echo '<--------------- Docker Publish Started --------------->'  
                docker.withRegistry(registry, 'cb52d53b-89d7-4192-b1e4-a625fed0979a'){
                    app.push()
                }    
               echo '<--------------- Docker Publish Ended --------------->'  
            }
        }
    }
        //  stage(" Deploy ") {
        //   steps {
        //     script {
        //        echo '<--------------- Deploy Started --------------->'
        //        sh 'helm install twittertrend-2.0.2 ttrend'
        //        echo '<--------------- Deploy Ends --------------->'
        //     }
        //   }
        // }    
    }
    }
