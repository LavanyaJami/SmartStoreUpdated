pipeline {
        agent {label 'windows'}
        parameters {
                booleanParam(name: 'Run_Build', defaultValue: false, description: 'Will Build Code and execute Test Cases')
                booleanParam(name: 'Run_SonarAnalysis', defaultValue: false, description: 'Will Run Sonar Code Analysis')
                booleanParam(name: 'Release', defaultValue: false, description: 'Will Push code to the Server')             
        }
        tools {
                   jdk 'PratianTFS'
                   //maven 'maven'
                   //msbuild 'msbuild15'
                   //sonarqube scanner 'sonarscanner'
               }
        stages {
                stage('checkout'){
                        
                        steps {
                                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], gitTool: 'default', submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/vmadykumar/SmartStoreUpdated.git']]])
                        }
                }
                stage('build') {
                      steps {
                                       echo 'Hello ASP.NET, Executing build'
                                       bat 'ClickToBuild.cmd -Dbuild.number=${BUILD_NUMBER}' 
                       }                      
                }
                stage('Test'){
                      steps {
                                       echo 'Executing Test Cases'
                                       bat 'ClickToTest.cmd'
                      }
                }
                stage('sonar analysis') {
                        when {
                                expression { params.Run_SonarAnalysis == true}
                        }
                        steps { 
                                
                                        echo 'Executing sonar Analysis'        
                                        withSonarQubeEnv('sonarscanner') {
                                                bat 'sonar:sonar'  
                                        
                                }
                        }
                }
                //stage('Artifactory'){
                  //      steps { 
                                
                  //                      echo 'creating Artifacts'         
                  //                      archiveArtifacts 'build/*.war'
                  //                      echo 'Artifact created'
                                
                  //       }
                 //}
                stage('Approval Step'){
                                   when {
                                        expression { params.Release == true}
                                   }
                                   steps{
                                           script {  
                                                input message: 'Approve Deploy?', ok: 'Yes', submitter: 'PME'
                                               // echo "${env.approved}"
                                                // echo ("User Input is: "+env.approved['env'])
                                                }
                                   }
                }
                stage('Release'){
                                agent {label 'windows'}
                                when {
                                        expression { params.Release == true }
                                }
                        steps {
                                echo 'Starting Release'
                                deploy adapters: [tomcat7(credentialsId: '51d79b50-5fd8-4444-91eb-c6ead7dc4151', path: '', url: 'http://172.30.13.143:8081')], contextPath: '/HappyTrip', war: 'Code/target/*.war'
                                echo 'Release Completed'
                        }
                }
                //stage('Notification') {
                //        steps {
                //                echo 'Sending Email'
                //                emailext body: 'Build Suceeded', subject: 'Jenkins Job Status', to: 'vikash.bcet@gmail.com'
                //        }
                //}
        }
        post {
                        always {
                                echo 'One way or another, I have finished'
                                deleteDir() /* clean up our workspace */
                        }
                        success {
                                echo 'Yeppie.. I succeeeded!'
                                emailext body: 'Everything went well :) ${env.BUILD_URL}', subject: 'Succeeded Pipeline: ${currentBuild.fullDisplayName}', to: 'vikash.bcet@gmail.com'
                        }
                        unstable {
                                echo 'Oh! No I am unstable :/'
                        }
                        failure {
                                echo 'Shit.. I failed :('
                                emailext body: 'Something is wrong with ${env.BUILD_URL}', subject: 'Failed Pipeline: ${currentBuild.fullDisplayName}', to: 'vikash.bcet@gmail.com'
                        }
                        changed {
                                echo 'Things were different before...'
                        }
        }
}
