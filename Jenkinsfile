pipeline {
@Library('middletier@optumdevops/midtierpatientjenkinspipeline.git') _
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
                parallel(
                          environment:{
                             echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"

                             script{
                                      def pom = readMavenPom file: 'pom.xml'
                                      echo sh(script: 'env', returnStdout: true)
                                 }
                           },
                           verifyMaven:{
                             sh '''
                                 echo "PATH = ${PATH}"
                                 echo "M2_HOME = ${M2_HOME}"

                             '''
                           }
                         )
                      }

              }

        stage ('Build') {

            steps {
                sh 'mvn -U clean install'

              }
            }

            stage('Junit') {
             when {
               expression {
                 currentBuild.result == null || currentBuild.result == 'SUCCESS'
               }
             }
             steps {
               parallel(
                         JunitTest:{
                          junit 'target/surefire-reports/**/*.xml'
                          },

                        )
                     }
                   }


          stage('archiveArtifacts'){
          when {
                expression {
                          currentBuild.result == null || currentBuild.result == 'SUCCESS'
                        }
                }
                steps {
                    parallel(

                      archive:{
                        archiveArtifacts artifacts: '**/*', fingerprint: true
                      }
                      )
                 }
            }
          stage('SONAR PHASE') {
           when {
             expression {
               currentBuild.result == null || currentBuild.result == 'SUCCESS'
             }
           }
           steps {

             parallel(
                      jacocomavenplugin:{
                        script{
                              sh 'mvn -U clean org.jacoco:jacoco-maven-plugin:0.7.2.201409121644:prepare-agent install -Dmaven.test.failure.ignore=true'
                              }
                               },
                        SonarShellscript:{

                            sh '''
                              git clone https://msmit198@codehub.optum.com/optumdevops/midtierpatientjenkinspipeline.git
                              cd midtierpatientjenkinspipeline
                              chmod 777 *
                              ./sonar.sh

                            '''
                          }

                        )
                     }
                 }


                  stage ('Artifactor Deployment') {
                    when {
                      expression {
                        currentBuild.result == null || currentBuild.result == 'SUCCESS'
                      }
                    }
                     steps {
                       parallel(
                            Push_code_t0_artifactory: {
                                  echo "past code here"
                                  },

                              )
                            }
                         }
                         stage ('OC App Deployment ') {
                           when {
                             expression {
                               currentBuild.result == null || currentBuild.result == 'SUCCESS'
                             }
                           }
                           environment {
                           OCP_MYSQL_DB_PASSWORD     = credentials('ce281be0-13d5-469f-8e2a-f3739e21cbaf')
                           OCP_MYSQL_DB_USERNAME = credentials('ocpm_own_stg')
}
                            steps {
                              parallel(
                                    openshift_projects: {
                                    sh '''

                                    chmod 777 midtierpatientjenkinspipeline/osedeploymentdev.sh
                                    ./midtierpatientjenkinspipeline/osedeploymentdev.sh

                                    '''
                                     },
                                     notify:{
                                       post {
                                         success {
                                           echo "I succeeded"

                                         }
                                         }                                    }

                                     )
                                   }
                                }
                            }
}
