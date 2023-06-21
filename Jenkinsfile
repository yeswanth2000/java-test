pipeline {
    agent any

    stages {

        stage('Build') {
            steps {
                echo 'Build'
            }
        }

        stage('Test') {
            steps {
               echo 'Test' 
            }
        }

        stage('Sonar Scanning') {
            steps {
                echo 'Scan'
            }
        }

        stage('Publish to Artifacory') {
            steps {
                 echo 'Artifact'
            }
        }

        stage('Deploy to Dev') {
            steps {
                echo 'Dev'
            }
        }

        stage('Deploy to Test') {
            steps {
                 echo 'Test'
            }
        }

        stage('Deploy to UAT') {
            steps {
                echo 'UAT'
            }
        }

        stage('Deploy to Stage') {
            steps {
               echo 'Stage' 
            }
        }

        stage('Deploy to Prod') {
            steps {
              echo 'Prod'   
            }
        }


    }
}