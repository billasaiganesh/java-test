pipeline {
    agent any

    environment {
        function_name = 'java-test'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Build'
                sh 'mvn package'
            }
        }

        stage('Sonar Qube Analysis'){
            agent any
            when{
                anyOf{
                    branch  'feature/*'
                    branch 'main'
                }
            }
            steps{
               withSonarQubeEnv('sonar'){
                sh  'mvn sonar:sonar'
            }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    try {
                        timeout(time: 10, unit: 'MINUTES') {
                            waitForQualityGate abortPipeline: true
                        }
                    }
                    catch (Exception ex) {

                    }
                }
            }
        }

        stage('Push') {
            steps {
                echo 'Pushed'

                sh "aws s3 cp target/sample-1.0.3.jar s3://s3-java-samples"
            }
        }
        //CI ended 
        //CD started 

        stage('Deployment'){
            parallel{
                stage('Deploy to PROD') {
                    steps {
                        echo 'Build'

                        sh "aws lambda update-function-code --function-name $function_name --s3-bucket s3-java-samples --s3-key sample-1.0.3.jar --region us-east-1"
            }
        }

            stage('Deploy to Test') {
                 when{
                    branch 'main'
                 }
                 steps {
                     echo 'Build'

                     sh "aws lambda update-function-code --function-name $function_name --s3-bucket s3-java-samples --s3-key sample-1.0.3.jar --region us-east-1"
                  }
                }
            }
        }



    }
}