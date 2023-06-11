pipeline {
    agent any

     options {
        //Disable concurrentbuilds for the same job
        disableConcurrentBuilds()
        // Colorize the console log
        ansiColor("xterm")          
        // Add timestamps to console log
        timestamps()
        
    }

    environment {
        // AWS_ACCESS_KEY = credentials('aws_access_key')
        // AWS_SECRET_KEY = credentials('aws_secret_key')
        ARTIFACTID = readMavenPom().getArtifactId()
        VERSION = readMavenPom().getVersion()
        S3_BUCKET = 'raghu-jenkinsartifacts'
        LAMBDA_FUNCTION = 'test'
    }

    stages {
        stage('Build') {
            steps {
                script {
                    echo 'Build'
                    echo 'ArtifactId: ${ARTIFACTID}'
                    sh 'mvn package -Dmaven.test.skip=true'
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    echo 'Test'
                    sh 'mvn test'
                }
            }
        }

        stage('Push to artifactory') {
            steps {
                script {
                    echo 'Push to artifactory'
                    def JARNAME = ${ARTIFACTID}+'-'+${VERSION}+'.jar'
                    echo "JARNAME: ${JARNAME}"          

                    sh "aws s3 cp target/${JARNAME} s3://$S3_BUCKET"
                }
            }
        }

        stage('Deploy to Lambda - Test') {
            agent any
            steps {
                script {
                    echo 'Deploy to Test'


                    // sh "aws lambda update-function-code --function-name $LAMBDA_FUNCTION  --zip-file fileb://target/${JARNAME}"
                }          
            }
        }

        stage('Release to Prod') {
            agent none
            steps {
                echo 'Release to Prod'
                script {
                    if (env.BRANCH_NAME == "main") {
                        timeout(time: 1, unit: 'HOURS') {
                            input('Proceed for Prod  ?')
                        }
                    }
                }

            }
        }

         stage('Deploy to Prod') {
            agent any
            steps {
                script {
                    if (env.BRANCH_NAME == "master") {
                        echo 'Deploy to Prod'

                        JARNAME = ARTIFACTID+'-'+VERSION+'.jar'

                        sh "aws s3 cp target/${JARNAME} s3://$S3_BUCKET/"
                        

                       // sh "aws lambda update-function-code --function-name $LAMBDA_FUNCTION --s3-bucket $S3_BUCKET --s3-key ${JARNAME}" 
                    }
                }
            }
        }

    }

    post {
      failure {
        echo 'failed'
        echo '${env.BUILD_NUMBER}'
          
         // can send notifications
      }
      success {
        echo 'Success'
      }
      aborted {
        echo 'aborted'
      }
    }
}
