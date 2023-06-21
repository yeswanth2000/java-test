def checkoutSourceCode() {
    checkout([$class: 'GitSCM', branches: [[name: "refs/remotes/origin/main"]], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'raghu-github', url: 'https://github.com/Raghupatik/java-test.git']]])
}

pipeline {
    agent any

     options {
        //Disable concurrentbuilds for the same job
        disableConcurrentBuilds()         
        // Add timestamps to console log
        timestamps()

        skipDefaultCheckout true  
    }

    parameters {
	    string(name: 'DeployVersion', defaultValue: '', description: 'Input the version to be deployed')
    }

    environment {
        S3_BUCKET = 'raghu-jenkinsartifacts'
        LAMBDA_FUNCTION = 'java-sample-lambq2'
    }

    stages {
        stage ('Checkout') {
	        agent { label 'jenkins-slave' }
            steps {
                checkoutSourceCode()
	        }
        }

        stage('Build') {
            agent { label 'jenkins-slave' }
            steps {
                script {
                    echo 'Build'
                    sh 'mvn clean install'
                }
            }
        }

        stage('Push to S3') {
            agent { label 'jenkins-slave' }
            steps {
                script {
                    echo 'Push to S3'
		            ARTIFACTID = readMavenPom().getArtifactId()
        	        VERSION = readMavenPom().getVersion()
                    JARNAME = ARTIFACTID+'-'+VERSION+'.jar'
                    echo "JARNAME: ${JARNAME}"    
	                //sh 'cp target/priceavailability-api*.war target/priceavailability-api.war'
                    withAWS(profile:'aviall', region:'us-east-1') {
				        s3Upload(file:"target/${JARNAME}", bucket:"${S3_BUCKET}", path:"${JARNAME}")
		            }

                    // sh "aws s3 cp target/${JARNAME} s3://$S3_BUCKET"
                    currentBuild.result = 'SUCCESS'
                }
            }
        }

        stage ('Run postman test on Production') {
            parallel {
                stage ('Run postman test on Tomcat Production') {
                    agent { label 'master'}
                    steps {
                        checkoutSourceCode()
                        script {
                            retry(2) {
                                try {
                                    echo 'Run test on staging server'
                                    sh 'newman run src/test/java/com/example/resources/postman-testscript.json -d src/test/java/com/example/resources/tomcat-env.json -k'
                                    currentBuild.result = 'SUCCESS'
                                } catch(Exception err) {
                                    currentBuild.result = 'FAILURE'
                                }
                            }
                        }
                    }
                }
            }
        }

        stage('Deploy to Lambda - Test') {
            agent { label 'jenkins-slave' }
            steps {
                script {
                    echo 'Deploy to Test'

                    sh "aws lambda update-function-code --function-name $LAMBDA_FUNCTION --region us-east-1 --s3-bucket $S3_BUCKET --s3-key ${JARNAME}"
                }          
            }
        }

        stage('Release to Prod') {
            agent none
            steps {
                echo 'Release to Prod'
                script {
                    if (env.BRANCH_NAME == "main") {
                        timeout(time: 10) {
                            input("Approve for deployment version ${VERSION} on Production?")
                        }
                    }
                }

            }
        }

         stage('Deploy to Prod') {
            steps {
                script {
                    if (env.BRANCH_NAME == "main") {
                        echo 'Deploy to Prod'
                        retry(3) {
	            		    sleep 10
                        }
                       // sh "aws lambda update-function-code --function-name $LAMBDA_FUNCTION --s3-bucket $S3_BUCKET --s3-key ${JARNAME}" 
                    }
                }
            }
        }

         stage ('Notify when all stages are done') {
	        agent { label 'master' }
            steps {
                script {
                    if (currentBuild.result == 'SUCCESS') {
			            echo "The deployment process is done, new version is deployed successfully"
                        mail (to: 'ragupathi.kommidi@gmail.com', subject: "SUCCESS: The job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) is done, new version ${VERSION} is deployed successfully", body: "Please go to ${env.BUILD_URL} for the detail")
                    } else {
                        mail (to: 'ragupathi.kommidi@gmail.com', subject: "ERROR: The job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) failed, rollback to last stable version", body: "Please go to ${env.BUILD_URL} for the detail")
			            echo "ERROR: The job ${env.JOB_NAME} (${env.BUILD_NUMBER}) failed, rollback to last stable version"
                        sh 'exit 1'
                    }
                }
            }
        }

    }
}
