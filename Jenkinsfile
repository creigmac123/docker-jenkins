pipeline {

    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
      }
    tools {
        maven 'M3'

    }
    environment {
         BUILD_ID = "${BUILD_NUMBER}"

    }
    stages {
        stage ('Initialize') {
          steps {
                       echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"

                             script{
                                      def pom = readMavenPom file: 'pom.xml'
                                      echo sh(script: 'env', returnStdout: true)
                                 }
                      }

              }
            }

}
