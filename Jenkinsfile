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

        stage('Push') {
            steps {
                echo 'Pushed'

                sh "aws s3 cp target/sample-1.0.3.jar s3://s3-java-samples"
            }
        }

        stage('Deploy') {
            steps {
                echo 'Build'

                sh "aws lambda update-function-code --function-name $function_name --s3-bucket s3-java-samples --s3-key sample-1.0.3.jar --region us-east-1"
            }
        }
    }
}