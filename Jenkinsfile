pipeline {
    agent any

    environment {
        function_name = 'java-test'
    }

     parameters {
        string(name: 'RollbackVersion', description: 'Please enter rollback version')
        choice(
            choices: ['Dev', 'Test', 'Prod'],
            name: 'Environment',
            description : 'Please select environment'
        )
    }

    stages {
        stage('Build') {
            steps {
                echo 'Build'
                sh 'mvn package'
            }
        }

        stage('Sonar Qube Analysis') {
            agent any
            when {
                anyOf {
                    branch  'feature/*'
                    branch 'main'
                }
            }
            steps {
                withSonarQubeEnv('sonar') {
                    sh  'mvn sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
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

                sh 'aws s3 cp target/sample-1.0.3.jar s3://s3-java-samples'
            }
        }
        //CI ended
        //CD started

        stage('Deployment') {
            parallel {
                stage('Deploy to Dev') {
                    function name = java-dev-env
                    steps {
                        echo 'Build'

                        sh "aws lambda update-function-code --function-name $function_name --s3-bucket s3-java-samples --s3-key sample-1.0.3.jar --region us-east-1"
                    }
                }

                stage('Deploy to Test') {
                    function name = java-test-env

                    steps {
                        echo 'Build'

                        sh "aws lambda update-function-code --function-name $function_name --s3-bucket s3-java-samples --s3-key sample-1.0.3.jar --region us-east-1"
                    }
                }
            }
        }


        stage('Deployement to Prod') {
            when {
                expression { return params.Environment == 'Prod'}
            }
            steps{
                input(
                    message: "Are we good to go for deployment?"
                )
            }
           
        }

        stage('Release to Prod'){
            when{
                branch "main"
            }
             steps{
                 sh "aws lambda update-function-code --function-name $function_name --s3-bucket s3-java-samples --s3-key sample-1.0.3.jar --region us-east-1"
            }
        }

    }
        post {
        always {
            echo "${env.BUILD_ID}"
            echo "${BRANCH_NAME}"
            echo "${JENKINS_URL}"

        }

        failure {
            echo 'failed'
        }
        aborted {
            echo 'aborted'
        }
    }
}

